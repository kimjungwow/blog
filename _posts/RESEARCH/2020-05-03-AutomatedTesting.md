---
title: "Research :: AutomatedTesting"
date: 2020-05-03 15:45:00 +0900
categories: Study
tags: Study Testing
---
KAIST CS453 Spring 2020 정리  

# 1 : Fundamentals of Testing

- Software testing : investigate program's **quality**
- Types of quality 
  - **Dependability**
    - correctness : 논리적 언어로 명시되고 증명 가능한 경우 SW가 올바르게 작동함
    - Reliability : 일정 기간 동안 높은 확률(통계학적)로 올바르게 동작해야 함.
    - Safety : 생명/자본의 위험 없어야 함
    - Robustness : 주변 환경이 변해도 SW가 dependable해야함
  - **Performance**
    - 실행 시간, 네트워크 처리량, 메모리 사용량 등
    - 실제 상황의 input을 모르기 때문에 테스트하기 어려움
  - **Usability**
    - 사용하기 쉬운가?
    - A/B testing, beta-testing 등
  - 자동화하기에는 dependability, reliability가 적합함. performance, usability, security 등은 힘듦
- Faults, Error, Failure
  - Faults : 에러로 이어질 **수 있는** 코드 속 오류. 같은 코드여도 fault를 어떻게 정의하냐에 따라 어떻게 고칠지가 달라짐
  - Error : fault의 실행 결과이자 failure로 이어질 **수 있음**
  - Failure : 프로그램 외부적으로 에러가 명백히 드러남
  - Fault가 있어도 실행하지 않으면 failure X
  - Error가 있어도 (내부에서 뭔가 잘못되도) 유저가 보지 못하면 failure X
  - 유저가 잘못된 것을 보면 failure

- Dynamic vs Static
  - Dynamic analysis - 프로그램을 실행해 잘 작동하는지 확인
  - Static analysis - 프로그램 실행 없이 correctness 확인

- Terminology
  - Test Oracle : test input에 대해 실행하였을 때 기대한 대로 실행되었는지 확인하는 메커니즘
    - System crash, unintened infinite loop과 같이 잘못되었음을 계산 없이 쉽게 알 수 있는 implicit oracles는 일부이다.
  - Test case : Test input + Test oracle
  - Test suite : a collection of test cases

- Dijkstra Quote
  - 테스팅은 버그의 존재만을 증명할 수 있고, 버그의 부재는 증명할 수 없다


- Black-box(tester doesn't look at the code) vs White-box(tester does)
- Testing Techniques
  - Random testing 
    - can be both black-box or white-box
    - randomly selected test inputs
    - 구현하기 쉽고, 실제 fault 발견 가능하지만, 오랜 시간동안 소득 없을 수도
  - Combinatorial Testing : how to combine inputs
    - black-box
    - Tester는 오직 input specification만을 앎
  - Structural Testing
    - White-box
    - 소스 코드의 structural unit을 통해 테스팅의 정확성을 파악
    - increasing coverage is necessary but not sufficient
  - Mutation Testing
    - White-box
    - huge potential but not without challenges
  - Regression Testing
    - can be both
    - 수정한 사항이 기존의 기능을 깨지 않았는지 확인

# 2 : Black Box Testing & Combinatorial Interaction Testing

- Black Box Testing
  - 코드 내부는 보지 않고, 프로그램의 specification만을 이용해 테스팅
  - Equivalence Partitioning
    - 프로그램은 같은 equivalence class안의 element들에 대해서는 똑같이 행동하므로, 각 class에서 한 개만 고름
  - Boundary Value analysis
    - 프로그래머는 주로 boundary of equivalence classes에서 실수를 저지르므로, boundary 근처에서 sample을 정함. Equivalence Partitioning과 함께 쓰임
  - Category Partition
    - 각 input 혹은 configuration을 하나의 category로 생각한 뒤, partition

- Combinatorial Interaction Testing
  - t(strength)개의 파라미터간의 모든 interaction을 포함해서 테스팅한다. 이를 covering array라고 부른다.
  - strength가 높으면 더 많은 input을 다루므로 fault를 더욱 잘 발견한다.

- IPOG with strength t
  - parameter 1~t에 대하여 모든 조합을 ts에 써넣음.
  - i = t+1~n일 때, parameter 1~i-1 중 t-1개의 조합+parameter i의 값을 합친 t개로 이루어진 조합들을 set PHI에 써놓음. 그 후, PHI의 내용을 가장 많이 지울 수 있는 것부터 ts에 추가해나감
- metaheuristic : IPOG보다 강력하지만 시간 오래 걸림

- Constraints
  - hard constraints와 soft constraints
  - constraint를 통해 CIT의 사이즈를 줄일 수 있음. 모든 constraint를 잘 조절하면 t=4 이상을 사용해도 좋음

# 3 : Testing FSM

- Finite State Machine (FSM) : finite set of states, initial state, input/output set, state transfer/output function
  - initially connected : initial state에서 모든 state로 갈 수 있음
  - strongly connected : 모든 점에서 다른 모든 점으로 갈 수 있음
  - equivalent : input sequence에 대해 같은 output sequence 냄
  - minimal : 더 state가 적은 equivalent FSM이 없음. minimal이 아니면 equivalent minimal FSM으로 바꿀 수 있음
  - Reset operation : inital state로 바꾸는 operation
  - 보통 reliable reset operation, minimal, strongly connected and completely specified 가정

- Faults
    - Output fault : transition has wrong output
      - Transition tour method : 모든 transition을 지나는 input sequence(꼭 최단 경로 아니어도 됨)를 실행해 output 확인. transfer fault 탐지는 보장 안 됨
      
    - State transfer fault : transition goes to the wrong state
      - 우선 output만을 이용해 현재 원하는 state인지 확인할 수 있어야 함
      - distinguishing sequence : 모든 state에 대해 다른 output sequence를 냄. 모든 FSM에 distinguishing sequence 있지는 않음
      - unique input/output sequence : 특정 state에서만 유일한 output sequence을 내는 input sequence
      - Characterising Set : 임의의 두 state를 구분하는 input sequence를 원소로 무조건 포함하는 집합
      - Chow's method : source state까지 잘 가는지 + 거기서 transition 하면 target state로 잘 가는지 확인. input set X, characterising set W, state cover set V(initial state->각 state 경로 모은 집합), concatenation operator ·에 대해 `Resulting Test Set= V·W U V·X·W`. 이 때 다른 것의 prefix인 것은 삭제 가능.

    - Fault 있으면 minimal이 아님

# 4 : Control and Data Flow

- Control Flow : 각 structural element가 실행되고 평가되는 순서
- Control Flow Graph : statements가 node, possible flow of execution이 edges. 항상 explicit end node를 가정함
- 모든 test cases를 다 실행하거나, 모든 fault를 다 발견하는 것은 실행 불가능(infeasible). 그래서 두 testcase를 비교할 때  additional test execution의 benefit을 따짐으로써 어떤 것을 실행할 지 정함
- Fault detection Capability
  - Structural Code coverage : necessary, but not sufficient. 어떤 것이 테스트 안 되고 있는 지는 알려주지만, 실제로 어떤 것이 테스트되고 있는지는 정확히 못 알려줌. 

- CFG
  - Simple path : 두 번 이상 통과되는 edge X
  - All possible path가 statement coverage보다 강력
  - 각 loop의 iteration 횟수를 제한. n개의 연속된 loop의 iteration 횟수 제한이 각각 k이면, `(k+1)^n`개의 path

- Decision Coverage : 전체 predicate의 value true/false 모두 다뤄짐
- Condition Coverage : 각 boolean subexpression의 value true/false 모두 다뤄짐
- Modified condition / Decision Coverage : 각 boolean subexpression의 value true/false 모두 다뤄지며, 각자가 final decision에 영향을 끼침

- Structural coverage는 대부분 CFG에 관한 것이었음. 반면 특정 값이 failure에 영향을 끼치는지 확인하는 **Dataflow analysis**. 기본적으로 CFG를 이용해 확인함

- 종류
  - d : value of variable defined
  - u_p : variable used in predicate
  - u_c : variable used in calculation
  - k : killed (undefined or memory released)
- data flow 패턴
  - d,d : harmless but suspicious
  - d,u : normal
  - d,k : harmless but suspicious
  - u,d : u앞에 d가 없으면 potentially suspicious
  - u,u : normal
  - u,k : normal
  - k,d : potentially suspicious
  - k,u : bug
  - k,k : suspicious
- DU paths
  - path from x to y is definition clear for variable v : x부터 y까지 중 v에 대한 assignment X
  - du-path : x에서 define, y에서 use, 그 중간에서 redefine X이며 simple path
- Use path : 하나의 d-u에서는 하나의 path만 선택.

- Mersuring coverage : additional code를 넣어 coverage 측정되도록 함

# 5 : Random Testing

- 개발자는 편견 때문에 잘 작동하는 테스트만 만들 수 있음

- 컴퓨터는 pseudo-random number generator 사용해서, seed에 의해 영향 받기 때문에 진짜 랜덤은 아니지만 주기가 길기 때문에 상관 없음. 진짜 랜덤을 원하면 HW 때문

- 확률 분포 : 예를 들어 큰 input이 더 에러를 잘 유도하면 큰 input 위주로. uniform probability distribution도 하나의 방법.
  - Complex structure : tree도 random bits로 생각하면, 트리 안에 트리가 있는 것을 구현하기 어려움. Ad-hoc generator approach를 통해 growing data-structure를 랜덤으로 만듦
  - Memory/Time constraints : 무한히 큰 input 다룰 수 없기 때문에 제한 필요. 하지만 어떤 크기의 input이 에러와 관련 있는지에 따라 constraints를 잘 조절해야 함
  - Test Input Length : binary string에서 길이 L인 string의 개수가 길이가 L미만인 string 개수보다 많기 때문에, 랜덤으로 길이를 구한 뒤 해당 길이의 string을 랜덤으로 정함                                                                             
단점
  - No guidance : needle in haystack. 전체 input 중 우리가 필요로 하는 조건 만족하는 input 아주 적음 → random value 대신 코드에 있는 값, 이전에 성공한 값 등을 넣음
  - Oracle : random input에 대해 oracle을 구할 수 없음 (외부 library 이용하는 것도 방법) → reference system 혹은 pre&post condition 이용

- Diversity : test input끼리 떨어져 있어야 faulty region에 들어갈 확률 큼. 이 때 distance를 정의하기 어려울 수 있음.
  - Euclidean distance
  - Hamming distance (edit distance) : 두 string간에는 서로 다른 글자 수

- Adaptive Random Testing : 새 input을 추가할 때는, Z개의 후보들 중 기존의 input들에 가장 diverse인 것을 추가. 이 때, 총 k개의 테스트 케이스를 구하려면 `|Z|k(k-1))/2`번 distance 계산해야하는데, k의 값에 따라 너무 클 수 있음.

- Ideal sampling under 1D : 어느 프로그램이 길이가 `|z|`인 1개의 faulty region 가질 때, input간의 거리 및 양 끝점과 input의 거리의 최소값이 z이상이면 failure 발견할 확률이 최대. 이와 같이 input 위치 정하는 것을 deterministic placement(DT)라 할 때, faulty detection이 100%되기 까지 필요한 testcase의 개수는 DT, ART, Random Testing 순으로 작다

- Output Diversity : 지금까지 본 적 없는 output을 내는 테스트는 주목해야 함. example-based testing보다 강력할 수도.

- Property Based Testing
  - input-output pair oracle이 아닌, property based oracles

# 6 : Structural Testing

- Structural adequacy criteria는 necessary, not sufficient

- **자동적으로** branch/statement/All path coverage를 달성하는 방법은?

- Path Condition
  - 특정 경로로 프로그램이 실행되게 하는 predicates의 모음
  - Search-Based Testing과 Dynamic Symbolic Execution의 기본

- Search-Based Testing
  - path condition을 fitness function으로 바꾼 뒤 meta-heuristic search를 이용해 fitness function을 최소화/최대화 함으로써 value 찾음
  - Fitness function for branch coverage = approach_level + normalise(branch_distance)
  - approach level : 타겟을 둘러싼 un-penetrated nesting levels의 수
  - branch distance의 예 : predicate `y>=x`에 대해 특정 K에 대하여 `x-y+K`를 최소화
  - branch_distance가 approach_level에 비해 너무 커질 수 있기 떄문에, 0과 1사이의 값으로 만들기 위해 normalise
  - Hill Climbing
    - 임의의 수로 시작해서 fitness 계산후, 이웃들에 대해서도 fitness 계산 한뒤 개선되는 방향으로 이동
  - Alternating Variable Method : hill climbing과는 다른 방법
    - exploratory move : 어느 방향으로 갈지
    - pattern move : 해당 방향으로 가속화하기 위해, branch distance 줄이는 방향으로 점점 많은 양 이동
  - Search-Based Testing은 non-structural critieria도 적용 가능. 하지만 branch distance의 개념이 어려우며, 실행 시간 긺

- DSE
  - constraint solver를 이용해 value 찾음
  - basic(random) input에서 시작하여 프로그램을 실행하며 path condition을 모음. 그 뒤 마지막 condition을 negate하며 계속 진행
  - Dynamic Symbolic Execution은 빠르지만 constraint solver(floating number 등은 잘 못 다룸)의 성능이 중요함.

- Whole Test Suite Generation
  - 기존의 test input을 변형하거나 섞어서 새 input을 만듦

# 7 : Mutation Testing

- White-box testing. 새로운 fault를 삽입함으로써, 기존의 test suite가 fault를 잘 탐지하는지 평가함.
  - Competent Programmer Hypothesis : 프로그래머의 버그는 아주 단순한 버그
  - Coupling Effect Hypothesis : 모든 단순한 fault를 해결할 수 있다면, 더 복잡한 fault들도 해결 가능
  - 그래서 단순한 fault들을 의도적으로 넣음

- 문법적으로는 달라도 의미적으로 같은 equivalent mutant 

- How to kill a mutant
  - reachability : should cover mutant
  - Infection : mutated code로 인해 state가 달라져야 함
  - Propagate : 달라진 state가 눈에 띄는 state로 보여야 함
  - Weak kill은 위 2개, strongly kill은 전부

- Subsuming Mutant : different mutants M1, M2에 대해 kill M1이면 kill M2인 경우, M1 subsumes M2

- compiler를 통해 같은 binary로 컴파일 되면 equivalent mutant로 판단 가능.

- High Order Mutant : mutation operator을 2번 이상 적용함. First Order Mutant보다 죽이기 어려움

# 8 : Regression Testing

- SW의 새로운 feature 혹은 수정된 사항 때문에 기존의 기능이 망가지는 경우 Regression fault 발생
- 새로운 버전이 잘 동작하는지 확인할 때, 이전의 모든 테스트를 다시 해보는 Retest-all 대신 아래의 세 가지를 함
  - Test Suite Minimisation
  - Regression Test-case Selection
  - Test Case Prioritisation

## Test Suite Minimisation

- **redundant** test를 없애서 regression test suite 최소화
- Greedy Minimisation
  - 가장 많은 editorial criteria를 다루는 testcase부터 선택
  - 각 testcase의 cost가 다르면, coverage/cost 큰 것부터 선택
- Multi-Objectiveness : Coverage외에 execution time, fault coverage 등의 다른 것도 따져야하는 경우

## Regression Test-case Selection
- SW의 변화된 부분을 실행하는 테스트만 선택함
- Textual Difference : 변화된 줄만 실행
- Control Flow Graph(CFG) 이용 : 그래프의 변화된 지점을 지나는 테스트만 실행
- 그 외에도 Design artefacts(UML), data-flow analysis, path analysis, symbol execution 등 다양한 테크닉을 통해 테스트 선택 가능
- **Safe** Selection : 최근 수정에 관련된 fault를 가질 가능성이 있는(remotest possibility) 모든 테스트를 실행. remotest possibility라 함은 테스트가 프로그램의 수정된 부분을 실행하지만, 그 이상은 보장받지 못함
- 단점 
  - executing trace (각 테스트가 어떤 것을 실행하는지) 구하기 어려움
  - configuration 등 non-executable modifications
  - Safe selection을 모두 실행하는 데 너무 오래 걸리는 경우 (expensive)

## Test Case Prioritisation
- 중요한 테스트를 먼저 실행해서 멈췄을 때 regression testing이 최대한 진행되어있도록 함
- 테스트 케이스 만들 때마다 1~100으로 이름 붙이면, 1~100보다는 랜덤 혹은 100~1이 더 좋음
- 어떤 테스트가 어떤 fault를 발견할 지 미리 알 수 없음. 그래서 surrogate(대리) 이용 - branch, block, etc의 surrogate를 가장 많이/가장 빨리 cover한다면 fault detection도 빠른 것으로 생각 가능
- Average Percentage of Fault Detection (APFD) : test suite fraction과 percent detected faults의 그래프의 아래 부분 넓이. 클 수록 좋음. coverage/execution time 큰 테스트를 먼저 해야 이득

## Continuous Integration

- 개발자가 변경 사항 commit 하기 전에 local machine에서 테스트 : pre-commit testing
- commit된 후 CI system이 자동적으로 post-commit testing
- Cloud Computing을 통해 병렬적으로 실행하면 빨리 끝남
- commit이 아주 많은 경우 테스트 케이스를 prioritise 하지 않고, commit을 prioritise

# 9 : Fault Localisation

- 디버깅 과정은 Test input → Execution → Test Results → Locale Faults → Desing Patch → Apply Patch로 이루어짐.
  - Test input → Execution → Test Results 에서 문제되는 input 발견 가능
    - Delta debugging
  - Test results → Locale Faults에서 문제되는 코드 발견 가능
    - Information Retrieval Technique
    - Spectrum Based Fault Localisation
    - Mutation Based Fault Localisation

## Delta Debugging
- Minimise failure inducing inputs through binary search
- 문제가 되는 input이 두 개 이상의 separate parts인 경우 : Recursive Delta Debugging
- Input이 highly structured 인 경우, 단순히 자르면 input structure가 invalidated : Hierarchical Delta Debugging (handle structural elements)
  - source code : delete AST nodes
  - HTML : delete XML nodes

## Information Retrieval (IR) based FAult Localisation
- 소스 코드 : collection of documents
- 버그 : query
- 버그 리포트와 가장 match하는(거리가 가까운) 소스 코드 찾기
- Vector Space Model (VSM)로 버그와 소스코드를 벡터로 나타낸 뒤, 거리를 구함
  - 의미 있는 N개의 단어로 이루어진 vocabulary를 구함
  - 각 단어가 query 및 documents에 들어있는지 확인 - i번째 단어가 j 번째 document에 안 들어있으면 w_i,j=0, 들어있으면 **tf-idf**라는 non-zero number로 표현
  - tf-idf
    - tf(t,d) : t라는 단어가 document d에 들어있는 횟수
    - idf(t,D) : 전체 단어의 개수가 N일 때, log(N/t를 포함한 document의 개수)
      - t를 포함한 document의 개수가 적으면, 즉 t가 흔한 것이 아니면 idf가 큼
      - t가 흔하면 idf 작음
    - tf-idf=tf X idf
      - tf-idf(t,d,D)가 크면 t가 D에서 흔하지는 않지만 d에는 들어있다. (d에서 unique)
      - tf-idf(t,d,D)가 작다면 
        - t가 d에 자주 나타나지 않거나
        - t가 D에 너무 흔하거나
- 각 파일들에 대하여 tokenisation, remove punctuation 등을 진행한 뒤, 각 파일의 VSM과 버그 리포트의 VSM 간의 거리(cosine distance)를 구함. 이 때 가장 거리가 짧은 파일이 버그 리포트에 나타난 증상에 가장 주 원인
- 장점 : 단순하고 성능 좋다.
- 단점 : 좋은 버그리포트가 필요하며, document(파일. the unit of localisation)이 특정 사이즈가 되어야 한다. line-level IR의 경우 query의 각 line의 distance를 계산해야해서 너무 계산량 많음. 그래서 file-level IR함
 
## Spectrum Based Fault Localisation (SBFL)
- 가정 : faulty line을 실행하면 아마 failure 발생할 것이다.
- test results와 structural coverage 이용
- 프로그램을 테스트했을 때, spectrum values (ef, ep, nf, np)를 Risk Evaluation Formula에 넣어 계산하여 Ranking - rank 높을 수록 확인해야할 statement가 적음
  - e/n : execute/not execute
  - p/f : pass/failure
  - higher ef : execute 할 때마다 failure 발생
  - higher up : avoid to execute 할 때마다 pass
  - Risk Evaluation Formula
    - Tarnatula = (failing ratio)/( (pass ratio) + (failing ratio))
      - failing ratio = ef/(ef+nf)
      - passing ratio = ep/(ep+np)
    - 유교수님의 제안 : GP
      - 대체로 우수함. 일부 formula에게는 조금 밀리는 경우도 있지만, 매번 밀리는 것은 아님
    - Hybrid SBFL Approach
      - 하나의 formula만 사용하면 제한됨. 동시에 여러 formula를 사용하거나, additional input feature가지는 SBFL formula사용
    - 유교수님의 제안 : Faulty Localisation Using Code and Change Metrics (FLUCCS)
      - Age(얼마나 element가 오래 코드에 존재했는지)
      - Churn(얼마나 자주 element가 변경되었는지)
      - Complexity(얼마나 element가 복잡한지)
      - 3가지를 기준으로 테스트해보기 전에 fault가 어디있는지 확인 후, GP/SVM 이용하여 Rank
  - perfect bud understand assumption : 개발자는 faulty line보면 버그임을 알아 차릴 수 있다는 가정. 하지만 실제로는 봐도 모르기 때문에, SBFL에서 rank 높은 line 봐도 버그인지 모름. 그래서 faulty line을 보려면 정확히 몇 번째 줄을 봐야하는지 명시해야함
  - Crash Course into Our Proof System
    - 주어진 test suite와 formula R에 대해서, 프로그램을 나누면
      - S^R_B = faulty element보다 높은 랭크의 elements
      - S^R_F = faulty element와 같은 랭크의 elements
      - S^R_A = faulty element보다 낮은 랭크의 elements
    - Formula R1 dominates formula R2 when S^R1_B is in S^R2_B and S^R2_A is in S^R1_A : R1이 R2보다 랭크 낮은 것들 다 포함
    - 서로 dominate이면 equivalent
  - 장점 : 직관적이며, coverage와 test results만 필요로 함
  - 단점 : single formula로는 모든 fault 다루기 제한됨. omission fault는 못 다루고, multiple faults 는 잘 못 다룸

## Mutation Based Fault Localisation
- Mutate faulty program
  - if mutating correct statements
    - Pass(P), Failing(F) tests 개수 같다면 : equivalent mutant
    - P-, F+ : New fault
    - P+, F- : Mask
  - if mutating faulty statements
    - P,F 개수 같으면 : equivalent
    - P?, F? : New vault
    - P+, F- : Mask or partial fix
- fault statement 안의 대부분 statments들은 correct이므로, MBFL 통해 faulty one 찾을 수 있음. 각 statement마다 mutation을 적용했을 때 P와 F개수가 어떻게 변하는지 관찰함으로써

# 10 : Non-testable Programs and Metamorphic Testing

- Oracal Assumption : oracle은 program의 옳고 그름을 판단할 수 있고, orcale 없이는 program을 테스트 할 수 없다는 가정
  - oracle을 만들 수 없거나, 찾기 매우 힘들면 위 가정 위배됨. 이런 프로그램을 non-testable이라 함. 그 중 아직 우리가 풀지 못한 문제를 구하는 non-testable 프로그램을 다룰 것. 주로 패턴을 찾는 scientific computation.
  - 그 외에도 아주 많은 output을 만들거나 잘못 이해된 프로그램도 non-testable program임

- Metamorphic Testing
  - 프로그램 p와 metamorphic relationship f,g에 대하여 p(i)=r → p(f(i))=g(r)
  - 기존의 input-output pair에 metamorphic relationship를 적용하여 새로운 input-output pair 예측 가능
  - property-based testing처럼 어떤 input이 오는지에 따라 관계가 정해지는 program invariant와 다름.
    - 예 : metamorphic testing에서는 linear function에서 p((x1+x2)/2)=(y1+y2)/2이므로 program invariant가 아님
  - metamorphic relationship을 구하려면 우선 quadratic model로 예측. non-numeric인 경우 combinational model 확인
  - metamorphic testing을 이용해 input image를 바꿔 DNN에서 다르게 classified 되도록함
