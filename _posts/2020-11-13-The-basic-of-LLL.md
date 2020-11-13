---
layout: post
title: The basic of LLL
use_math: true
---

# Introduction Linear algebra to LLL Algorithm

## Basic Linear algebra

**Vector Spaces** : vector space $\mathbb{V}$ 라 함은  $ \alpha_1\mathbf{v}_1 + \alpha_2\textbf{v}_2 \in \mathbb{V} \mbox{ for all } \textbf{v}_1, \textbf{v}_2 \in \mathbb{V} \mbox{ and all } \alpha_1,\alpha_2 \in \mathbb{R}$ 을 만족하는 $\mathbb{R^m}$ 공간 위의 부분집합 $\mathbf{V}$ 를 말한다. 

**Linear Combinations** : $\mathbf{v}_1, \mathbf{v}_2, \ldots,\mathbf{v}_k \in \mathbb{V}$ 를 정의하자. $\mathbf{v}_1, \mathbf{v}_2, \ldots,\mathbf{v}_k \in \mathbb{V}$ 의 linear combination 이란$\mathbf{w}=\alpha_1\mathbf{v}_1+\alpha_2\mathbf{v_2} + \cdots +\alpha_k\mathbf{v}_k \mbox{ with } \alpha_1,\ldots\alpha_k \in \mathbb{R}$ 의 $\mathbf{w}$ 와 같은 형식의 벡터 를 말한다. 그리고 이러한 linear combination들의 조합 $\{\alpha_1\mathbf{w}_1 + \alpha_2\mathbf{w}_2+\cdots+\alpha_k\mathbf{v}_k  : \alpha_1,\ldots,\alpha_k \in \mathbb{R}\}$ 을 $\{\mathbf{v}_1,\cdots,\mathbf{v}_k\}$ 를 **span** 한다고 한다. 

**Independece** : 벡터들의 집합 $\mathbf{v}_1,\mathbf{v}_2,\ldots,\mathbf{v}_k \in \mathbb{V}$ 가 Linearly independent 할 조건은 $\alpha_1\mathbf{v}_1 + \alpha_2\mathbf{v}_2,+\cdots,+\alpha_k\mathbf{v}_k = \mathbf{0}$ 이 되는 해가 오직 $\alpha_1 = \alpha_2 = \cdots = \alpha_k = \mathbf{0}$ 로 유일하면 된다. 참고로 저 영벡터를 만드는 해가 $\alpha_1 = \alpha_2 = \ldots = \alpha_k = \mathbf{0}$ 말고 더 있으면 **Linearly dependent** 하다고 한다. 

**Basis** : 어떠한 vector space $\mathbb{V}$ 의 basis (기저) 라 함은 $\mathbf{V}$ 를 span하는 linearly independent vector들 $\mathbb{v}_1, \mathbb{v}_2,\ldots\mathbb{v}_n$ 의 집합을 말한다. 그리고 이러한 basis 들은 모든 벡터 $\mathbf{w} \in \mathbb{V}$ 가 특별한 해 $\alpha_1,\ldots,\alpha_n \in \mathbb{R}$ 에 대해 다음과 같은 형태로 쓸 수 있음을 말한다. $$\mathbf{w} = \alpha_1\mathbf{v}_1 + \alpha_2\mathbf{v}_2+\cdots+\alpha_n\mathbf{v}_n$$

**Dimension** : 어떠한 vector space $\mathbb{V} \in \mathbb{R}^m$ 를 정의하자.

- $$\mathbb{V}$$ 의 basis가 존재하고,
- 어떠한 $\mathbb{V}$ 의 두 basis가 동일한 수의 원소를 가지고 있다면, basis의 원소 갯수는 $\mathbb{V}$ 의 차원이라고 한다.

**Orthogonal** : 어떠한 $m$ 차원 실수공간 $\mathbb{R}^m$ 의 벡터공간 $\mathbf{V}$ 의 벡터 $\mathbb{u}, \mathbb{w}$ 를 정의하자. 이때 $\mathbb{u} = (x_1, x_2, \ldots, x_m), \mathbb{w} = (y_1, y_2, \ldots, y_m) $ 라고 하면, 두 벡터의 내적곱은 $\mathbb{u} \cdot \mathbb{w} = x_1y_1 + x_2y_2 + \ldots + x_my_m$  라고 하고, 두 내적곱의 결과가 0이면 두 벡터는 **orthogonal** (직교) 한다고 한다. 

**Norm** : $\mathbb{v}$ 의 길이, 혹은 euclidean norm은 다음과 같이 구할 수 있다. $$\| \mathbb{v}\| = \sqrt{x_1^2 + x_2^2 + \ldots + x_m^2}$$

**Orthogonal Basis** : 벡터 공간 $\mathbb{V}$ 의 Orthogonal Basis (정규직교기저) 라고 함은 벡터공간의 기저 $\mathbb{v_1}, \mathbb{v_2}, \ldots, \mathbb{v_n}$ 들이 서로 다음 조건을 만족하면 Orthogonal Basis 라고 한다. $$\mathbb{v}_i \cdot \mathbb{v}_j = \mathbb{0} \mbox{ for all } i \ne j.$$  

그리고 많은 선형 대수 이론들이 orthogonal 하거나 orthogonal basis 라고 가정을 한 상태에서 만들어진 공식들이 많다고 한다. 생각을 좀 해봤는데  orthogonal 하다고 가정하면 게산도 그렇고 여러모로 편한점이 많은 것 같다. 그래서 직교를 좋아하는 듯 zzzz. 

그리고 만약 $\mathbb{v}_1, \mathbb{v}_2, \ldots \mathbb{v}_n$ 가 정규직교기저고 $\mathbb{v} = \alpha_1\mathbb{v}_1 + \alpha_2\mathbb{v}_2 + \cdots + \alpha_n\mathbb{v}_n$ 가 기저들의 선형 결합으로 표현된다면, 

$\|\mathbb{v}\| = \|\ \alpha_!\mathbb{v}_1 + \cdots + \alpha_n\mathbb{v}_n\|^2 = (\alpha_1\mathbb{v}_1 + \cdots + \alpha_n\mathbb{v}_2) \cdot (\alpha_1\mathbb{v}_1 + \cdots \alpha_n\mathbb{v}_n) = \sum_{i=1}^{n} \sum_{j=1}^{n} \alpha_i\alpha_j(\mathbb{v}_i \cdot \mathbb{v}_j) = \sum_{i=1}^{n} \alpha_i^2\|\mathbb{v}_i\|^2 \mbox{ since } \mathbb{v}_i \cdot \mathbb{v}_j = 0 \mbox { for } i \ne j.$

그리고 basis 들이 정규직교기저면, $\|\mathbb{v}\| = \sum {a_i}^2$ 가 성립한다. 이걸 일반적인 방법으로 옮긴게 **Gram-Schmidt Algorithm** 이라고 한다.

이 알고리즘은 벡터공간에서 정규직교기저 를 생성해준다.

## Lattice

$m$ 차원 실수 공간의 basis vecvtor  $\mathbb{v}_1, \mathbb{v}_2, \cdots \mathbb{v}_n$ 을 정의하자. 이 때 **Lattice** $\mathbb{L}$ 이란  basis vector 들의 integer combination으로 만들어진 subspace 이다. 즉, 

 $$\mathbb{L} = \mathbb{L}(\mathbb{V}) = \{\sum_{i=1}^n c_i\mathbb{v}_i : c_i \in \mathbb{Z} \}, \mathbb{B} = (\mathbb{b}_1, \mathbb{b}_2, \ldots, \mathbb{b}_n) \mbox{ is basis of }\mathbb{L}$$ 이다. 

그리고 선형대수를 좀 배웠으면 이러한 선형 결합을 행렬로 바꿀 수 있다는건 잘 알 것이다. 이렇게 만든 행렬의 determinant는 꽤나 중요하게 다뤄진다고 한다. 그럼 이제 이 행렬의 성질을 조금씩 알아보자.

이 행렬의 rank가 full rank라고 가정하면,  

$$\det L = |\det(\mathbb{v}_1, \mathbb{v}_2,\ldots\mathbb{v}_n)|$$ 이다. 

**Hadamard’s inequality** : lattice $\mathbb{L}$ 의 determinant는 $\mathbb{L}$ 의 기저들의 노름끼리의 곱보다 항상 작거나 같다.

$$\det \mathbb{L} \le \prod_{i = 1}^n \|\mathbb{v}_i\|$$

## Shortest Vector Problem (SVP)

**Shortest Vector Problem (SVP)** 란 Lattice $\mathbb{L}$ 위에서 영점 $\mathbb{0}$ 과 가장 가까운 non-zero vector를 구하는 문제를 말한다.

이 SVP 문제가 중요한 이유는, 암호학적 시스템을 만드는 입장에서 생각 해보면 NP-hard로 증명된 문제이기 때문에 안전한 암호를 만들때도 도움이 되고, 양자컴퓨터에도 안전하다고 증명이 됐다. 공격하는 입장에서 보자면, 우리가 구하고 싶은 해를 shortest vector를 구하는 문제로 변형하면 SVP를 구하면 해를 구하는거나 마찬가지이다. 물론 실제 시스템에서 주어지는 키의 크기는 커서 컴퓨터로 구할 수 없을 정도지만, CTF에서는 키의 크기를 구할 수 있을 정도로 주는 경우가 많기 때문에 SVP를 푸는게 좋은 해답이 되는 경우가 많다.  그리고 이에 관한 유명한 정리도 하나 소개한다.

**Minkowski's first theorem** : lattice $\mathbb{L}$ 의 shortest vector 를 $\mathbb{v}$ 라고 하고, 차원을 ${D}$ 라고 하자. 그럼 다음과 같은 부등식이 성립한다.

$$\mathbb{v} \le \sqrt{D}|\det(\mathbb{L})|^{1/D}$$

이와 같은 정리를 소개하는 이유는, 가끔 SVP를 구해서 올바른 키를 찾은거같아도 안 되는 경우가 있는데, 그럴 때 위 부등식을 만족하는지 확인하면 좋다. 두 번째 이유는 그냥 신기해서....

## LLL (Lenstra–Lenstra–Lovász lattice basis reduction algorithm)







