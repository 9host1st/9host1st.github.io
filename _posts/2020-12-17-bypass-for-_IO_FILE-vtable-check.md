우분투 18.04 이상 버전에서는 우분투 16.04 와 달리 **IO_validate_vtable** 라는 함수로 vtable check를 하기 때문에 우분투 16.04에서의 일반적인 _IO_FILE vtable exploit을 할 수 없다. **IO_validate_vtable** 함수를 분석하고 우회하는 법을 알아보자.

## IO_validate_vtable

```c
IO_validate_vtable (const struct _IO_jump_t *vtable)
{
  /* Fast path: The vtable pointer is within the __libc_IO_vtables
     section.  */
  uintptr_t section_length = __stop___libc_IO_vtables - __start___libc_IO_vtables;
  uintptr_t ptr = (uintptr_t) vtable;
  uintptr_t offset = ptr - (uintptr_t) __start___libc_IO_vtables;
  if (__glibc_unlikely (offset >= section_length))
    /* The vtable pointer is not in the expected section.  Use the
       slow path, which will terminate the process if necessary.  */
    _IO_vtable_check ();
  return vtable;
}
```

먼저 _libc_IO_vtables의 섹션 크기를 구하고, 파일 함수가 호출될 때 사용하는 vtable이 _libc_IO_vtables에 존재하는지 검사한다. 만약 vtable 주소가 _libc_IO_vtable에 존재하지 않는다면, _IO_vtable_check 를 호출하여 포인터를 추가로 확인하게 된다. 즉 IO_validate_vtable를 우회하기 위해선

_libc_IO_vtable 섹션에 존재하는 함수만 사용해야 하는것이다. _libc_IO_vtable 섹션중에서 익스플로잇 하기 좋은 영역은 _IO_str_jumps 영역이 있는데, 다음과 같은 함수들이 존재한다.

```c
const struct _IO_jump_t _IO_str_jumps libio_vtable =
{
  JUMP_INIT_DUMMY,
  JUMP_INIT(finish, _IO_str_finish),
  JUMP_INIT(overflow, _IO_str_overflow),
  JUMP_INIT(underflow, _IO_str_underflow),
  JUMP_INIT(uflow, _IO_default_uflow),
  JUMP_INIT(pbackfail, _IO_str_pbackfail),
  JUMP_INIT(xsputn, _IO_default_xsputn),
  JUMP_INIT(xsgetn, _IO_default_xsgetn),
  JUMP_INIT(seekoff, _IO_str_seekoff),
  JUMP_INIT(seekpos, _IO_default_seekpos),
  JUMP_INIT(setbuf, _IO_default_setbuf),
  JUMP_INIT(sync, _IO_default_sync),
  JUMP_INIT(doallocate, _IO_default_doallocate),
  JUMP_INIT(read, _IO_default_read),
  JUMP_INIT(write, _IO_default_write),
  JUMP_INIT(seek, _IO_default_seek),
  JUMP_INIT(close, _IO_default_close),
  JUMP_INIT(stat, _IO_default_stat),
  JUMP_INIT(showmanyc, _IO_default_showmanyc),
  JUMP_INIT(imbue, _IO_default_imbue)
};
```

이 중에는 크게 두 가지의 함수를 쓸 수 있는데, **_IO_str_overflow** 와  **IO_str_finish** 이다. 이 두 가지 함수들을 이용한 공격 방법을 설명하겠다.

## _IO_str_overflow

```c
int _IO_str_overflow (_IO_FILE *fp, int c)
{
  int flush_only = c == EOF;
  _IO_size_t pos;
  if (fp->_flags & _IO_NO_WRITES)
      return flush_only ? 0 : EOF;
  if ((fp->_flags & _IO_TIED_PUT_GET) && !(fp->_flags & _IO_CURRENTLY_PUTTING))
    {
      fp->_flags |= _IO_CURRENTLY_PUTTING;
      fp->_IO_write_ptr = fp->_IO_read_ptr;
      fp->_IO_read_ptr = fp->_IO_read_end;
    }
  pos = fp->_IO_write_ptr - fp->_IO_write_base;
  if (pos >= (_IO_size_t) (_IO_blen (fp) + flush_only))
    {
      if (fp->_flags & _IO_USER_BUF) /* not allowed to enlarge */
	return EOF;
      else
	{
	  char *new_buf;
	  char *old_buf = fp->_IO_buf_base;
	  size_t old_blen = _IO_blen (fp);
	  _IO_size_t new_size = 2 * old_blen + 100;
	  if (new_size < old_blen)
	    return EOF;
	  new_buf = (char *) (*((_IO_strfile *) fp)->_s._allocate_buffer) (new_size);
```

크게 주목할 부분은 마지막 코드인데, _s._allocate_buffer 함수포인터를 실행시킨다. new_size를 인자로 전달하는데, 이 new_size를 계산해주는 코드를 보자.

```c
#define _IO_blen(fp) ((fp)->_IO_buf_end - (fp)->_IO_buf_base)
 
char *new_buf;
char *old_buf = fp->_IO_buf_base;
size_t old_blen = _IO_blen (fp);
_IO_size_t new_size = 2 * old_blen + 100;
 
if (new_size < old_blen)
   return EOF;
```

old_blen을 만들어주는 부분은 _IO_buf_end와 _IO_buf_base로 결정되는데, 이때 _IO_buf_end를 (some address - 100) / 2로 정해주고, _IO_buf_base를 0으로 조작해주면, new_size는 우리가 원하는 주소로 조작할 수 있다. 즉 인자를 조작할 수 있게된다. 이제 _s._allocate_buffer를 실행하기위한 조건을 확인해보자.

```c
int flush_only = c == EOF;
pos = fp->_IO_write_ptr - fp->_IO_write_base;
if (pos >= (_IO_size_t) (_IO_blen (fp) + flush_only))
    {
```

일단 flush_only는 0으로 정해져있고, pos 가 _IO_blen(fp), 즉 _IO_buf_end - _IO_buf_base보다 크면 되는데, 그냥 _IO_write_ptr - _IO_write_base >= _IO_buf_end - _IO_buf_base 이다. 이미 우변의 값은 정해져있으므로, _IO_write_ptr - _IO_write_base만 정해주면 된다. 이 값은 뭘로 하든지 상관이 없는것같다.

## _IO_str_overflow

```c
void _IO_str_finish (_IO_FILE *fp, int dummy)
{
  if (fp->_IO_buf_base && !(fp->_flags & _IO_USER_BUF))             
    (((_IO_strfile *) fp)->_s._free_buffer) (fp->_IO_buf_base);     
  
  fp->_IO_buf_base = NULL;
  _IO_default_finish (fp, 0);
}
```

매우 심플한 코드이다. 3번째줄의 조건문만 통과하면 되는데, fp->_IO_buf_base가 존재하면 된다. 그러면 _s._free_buffer에 system 함수 주소 넣고 _IO_buf_base에 /bin/sh 문자열 주소등을 넣으면 쉘을 획득할 수 있다.