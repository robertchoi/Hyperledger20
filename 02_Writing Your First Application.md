# 당신의 첫번째 어플리케이션 작성하기

~~~
노트
패브릭 네트워크의 기본 아키텍처에 아직 익숙하지 않은 경우 계속하기 전에 핵심 개념 섹션을 방문하십시오.

본 자습서는 패브릭 어플리케이션의 소개 역할을 하며 간단한 스마트 계약과 어플리케이션을 사용한다는 점도 주목할 가치가 있다. 패브릭 애플리케이션 및 스마트 계약에 대한 자세한 내용은 애플리케이션 개발 섹션 또는 상용 문서 튜토리얼을 참조하십시오.
~~~

이 튜토리얼에서는 패브릭 앱이 어떻게 작동하는지 보기 위해 몇 개의 샘플 프로그램을 살펴볼 것이다. 이러한 애플리케이션과 이들이 사용하는 스마트 계약은 일반적으로 FabCar라고 한다. 그들은 하이퍼레저 패브릭 블록체인(Hyperedger Fabric Blockchain)을 이해하기 위한 훌륭한 출발점을 제공한다. 응용 프로그램과 스마트 계약을 작성하여 원장을 쿼리하고 업데이트하는 방법과 인증 기관을 사용하여 허가된 블록체인과 상호 작용하는 응용 프로그램에서 사용하는 X.509 인증서를 생성하는 방법을 배우게 될 것이다.

우리는 애플리케이션 항목에서 자세히 기술된 애플리케이션 SDK를 사용하여 스마트 계약 SDK를 사용하여 원장을 쿼리하고 업데이트하는 스마트 계약을 호출할 것이다. 스마트 계약 프로세싱 섹션에 자세히 설명되어 있다.

다음 세 가지 기본 단계를 거친다.

1. 개발 환경 설정 우리의 어플리케이션은 상호작용을 할 네트워크가 필요하다. 그래서 우리는 우리의 스마트 계약과 어플리케이션이 사용할 기본적인 네트워크를 얻을 것이다.

2. 샘플 스마트 계약에 대해 배우는 것, FabCar. 우리는 자바스크립트로 작성된 스마트 계약을 사용한다. 우리는 스마트 계약을 검사하여 그 안에 있는 거래에 대해, 그리고 그 거래들이 어떻게 응용 프로그램이 원장을 조회하고 업데이트하는 데 사용되는지에 대해 배울 것이다.

3. FabCar를 사용하는 샘플 어플리케이션을 개발한다. 우리의 애플리케이션은 FabCar 스마트 계약을 사용하여 원장의 자동차 자산을 조회하고 업데이트할 것이다. 우리는 앱 코드와 그들이 만들어 내는 거래들, 즉 자동차 조회, 다양한 자동차 조회, 그리고 새로운 차 만들기를 포함한 것들의 코드로 들어갈 것이다.

이 튜토리얼을 완료한 후, 패브릭 네트워크의 피어에 호스팅되고 복제되는 원장과 상호 작용하기 위해 스마트 계약과 함께 응용 프로그램이 어떻게 프로그래밍되는지 기본적으로 이해해야 한다.

~~~
노트

이러한 애플리케이션은 서비스 검색 및 개인 데이터와도 호환되지만, 이러한 기능을 활용하기 위해 애플리케이션을 사용하는 방법을 명시적으로 보여주지는 않을 것이다.
~~~

# 블록체인 네트워크 구성
~~~
노트

다음 섹션에서는 패브릭 샘플 리포의 로컬 복제본 내의 첫 번째 네트워크 하위 디렉토리에 있어야 한다.
~~~
이미 Building Your First Network(첫 번째 네트워크 구축)를 실행했다면, 패브릭 샘플을 다운로드하고 네트워크를 가동하고 실행하게 될 것이다. 이 튜토리얼을 실행하기 전에 다음 네트워크를 중지하십시오.

~~~
./byfn.sh down
~~~
이전에 이 튜토리얼을 실행한 적이 있는 경우 다음 명령을 사용하여 오래된 또는 활성 용기를 제거하십시오. 참고, 이렇게 하면 패브릭과 관련이 있든 없든 모든 용기가 분해된다.

~~~
docker rm -f $(docker ps -aq)
docker rmi -f $(docker images | grep fabcar | awk '{print $3}')
~~~

네트워크 및 응용 프로그램에 대한 개발 환경과 함께 제공되는 아티팩트가 없는 경우 필수 구성 요소 페이지를 방문하여 시스템에 필요한 종속성이 설치되어 있는지 확인하십시오.

그런 다음 샘플, 바이너리 및 도커 이미지 설치 페이지를 방문하여 제공된 지침을 따르십시오. 패브릭 샘플 저장소를 복제하고 최신 안정적인 패브릭 이미지와 사용 가능한 유틸리티를 다운로드한 후 이 튜토리얼로 돌아가십시오.

Mac OS를 사용하고 Mojave를 실행하고 있다면 Xcode를 설치해야 한다.

# 네트워크 실행하기
~~~
노트

다음 섹션에서는 패브릭 샘플 리포의 로컬 복제본 내의 팹카 하위 디렉토리에 있어야 한다.
~~~

startFabric을 사용하여 네트워크를 시작하십시오.셸 대본 이 명령은 동료, 서명자, 인증 기관 등으로 구성된 블록체인 네트워크를 가동한다. 그것은 또한 원장에 접근하기 위해 우리의 어플리케이션에 의해 사용될 FabCar 스마트 계약의 Javascript 버전을 설치하고 인스턴스화 할 것이다. 튜토리얼을 통해 이러한 구성 요소에 대해 자세히 알아보십시오.

~~~
./startFabric.sh javascript
~~~

이제 샘플 네트워크를 가동하고 FabCar 스마트 계약을 설치하고 인스턴스화하십시오. 애플리케이션 사전 요구 사항을 설치하여 테스트해 보고 모든 것이 어떻게 작동하는지 확인해 봅시다.

# 어플리케이션 설치
~~~
노트

다음 지침은 패브릭 샘플 리포의 로컬 복제본 내의 팹카/자바스크립트 하위 디렉토리에 있어야 함을 요구한다.
~~~

다음 명령을 실행하여 응용 프로그램에 대한 패브릭 종속성을 설치하십시오. 완료하는 데 약 1분 정도 소요:

~~~
npm install
~~~
이 프로세스는 package.json에 정의된 핵심 애플리케이션 종속성을 설치하고 있다. 그 중 가장 중요한 것은 패브릭-네트워크 클래스인데, 애플리케이션은 신분, 지갑, 게이트웨이를 사용하여 채널에 연결하고, 거래를 제출하며, 통지를 기다릴 수 있다. 이 튜토리얼은 또한 패브릭-ca-client 클래스를 사용하여 사용자를 각각의 인증 기관에 등록하여 패브릭-네트워크 클래스 메소드에서 사용되는 유효한 ID를 생성한다.

npm 설치가 완료되면 애플리케이션을 실행할 수 있는 모든 것이 준비된다. 이 튜토리얼의 경우, 주로 팹카/자바스크립트 디렉토리에서 응용 프로그램 JavaScript 파일을 사용하십시오. 안에 무엇이 있는지 살펴보자.

~~~
ls
~~~

다음과 같은 화면을 볼 수 있다.
~~~
enrollAdmin.js  node_modules       package.json  registerUser.js
invoke.js       package-lock.json  query.js      wallet
~~~

예를 들어 팹카/타이프스크립트 디렉토리에 다른 프로그램 언어에 대한 파일이 있다. JavaScript 예를 사용한 후에는 이러한 내용을 읽을 수 있으며, 원칙은 동일하다.

Mac OS를 사용하고 Mojave를 실행하고 있다면 Xcode를 설치해야 한다.

# 관리자 사용자 등록
~~~
노트

다음 두 섹션은 인증 기관과의 통신을 포함한다. 새로운 터미널 셸을 열고 도커 로그 -f ca.example.com.를 실행하여 다음 프로그램을 실행할 때 CA 로그를 스트리밍하는 것이 유용할 수 있다.
~~~
네트워크를 만들 때, 말 그대로 admin이라고 불리는 admin 사용자가 CA(인증 기관)의 등록자로 생성되었다. 우리의 첫 번째 단계는 entry.js 프로그램을 사용하여 관리자를 위한 개인 키, 공용 키 및 X.509 인증서를 생성하는 것이다. 이 프로세스는 CSR(인증서 서명 요청)을 사용한다. 즉, 개인 및 공용 키가 먼저 로컬에서 생성되고 그 후에 공용 키가 CA로 전송되어 응용 프로그램에서 사용하기 위해 인코딩된 인증서를 반환한다. 그러면 이 세 가지 자격 증명이 지갑에 저장되어 우리가 CA의 관리자 역할을 할 수 있게 된다.

이후 우리는 새로운 애플리케이션 사용자를 등록하고 등록할 것이며, 이 사용자는 우리의 애플리케이션에서 블록체인과의 상호작용을 위해 사용될 것이다.

사용자 관리자를 등록하십시오.
~~~
node enrollAdmin.js
~~~

이 명령은 CA 관리자의 자격 증명을 지갑 디렉토리에 저장했다.

# 등록과 유저1 등록
이제 우리는 관리자 자격 증명을 지갑에 가지고 있으므로, 우리는 새 사용자인 user1을 등록할 수 있으며, user1은 원장을 쿼리하고 업데이트하는 데 사용될 것이다.

~~~
node registerUser.js
~~~
관리자 등록과 유사하게, 이 프로그램은 CSR을 사용하여 사용자1을 등록하고 관리자1과 함께 그 자격 증명을 지갑에 저장한다. 이제 admin과 user1이라는 두 개의 개별 사용자를 위한 ID가 있으며, 이러한 ID는 우리의 애플리케이션에 의해 사용된다.

원장과 교감할 시간...


# 원장 조회
블록체인 네트워크의 각 피어는 원장의 사본을 호스팅하며, 애플리케이션 프로그램은 원장의 최신 가치를 조회하여 애플리케이션에 반환하는 스마트 계약을 호출하여 원장을 조회할 수 있다.

다음은 쿼리의 작동 방식을 단순화한 것이다.

애플리케이션은 쿼리를 사용하여 원장의 데이터를 읽는다. 가장 일반적인 쿼리는 데이터의 현재 값인 세계 상태를 포함한다. 월드 상태는 키 값 쌍 집합으로 표시되며, 애플리케이션은 단일 키 또는 다중 키에 대한 데이터를 쿼리할 수 있다. 또한, 키 값을 JSON 데이터로 모델링할 때 복잡한 쿼리를 지원하는 CouchDB와 같은 데이터베이스를 사용하도록 원장의 월드 상태를 구성할 수 있다. 이는 특정 키워드와 특정 값이 일치하는 모든 자산(예: 특정 소유자가 있는 모든 자동차)을 찾을 때 매우 유용할 수 있다.

먼저, query.js 프로그램을 실행하여 원장에 있는 모든 차들의 목록을 반환하자. 이 프로그램은 두 번째 ID인 user1을 사용하여 원장에 액세스:

~~~
node query.js
~~~

출력은 다음과 같다.
~~~
Wallet path: ...fabric-samples/fabcar/javascript/wallet
Transaction has been evaluated, result is:
[{"Key":"CAR0", "Record":{"colour":"blue","make":"Toyota","model":"Prius","owner":"Tomoko"}},
{"Key":"CAR1", "Record":{"colour":"red","make":"Ford","model":"Mustang","owner":"Brad"}},
{"Key":"CAR2", "Record":{"colour":"green","make":"Hyundai","model":"Tucson","owner":"Jin Soo"}},
{"Key":"CAR3", "Record":{"colour":"yellow","make":"Volkswagen","model":"Passat","owner":"Max"}},
{"Key":"CAR4", "Record":{"colour":"black","make":"Tesla","model":"S","owner":"Adriana"}},
{"Key":"CAR5", "Record":{"colour":"purple","make":"Peugeot","model":"205","owner":"Michel"}},
{"Key":"CAR6", "Record":{"colour":"white","make":"Chery","model":"S22L","owner":"Aarav"}},
{"Key":"CAR7", "Record":{"colour":"violet","make":"Fiat","model":"Punto","owner":"Pari"}},
{"Key":"CAR8", "Record":{"colour":"indigo","make":"Tata","model":"Nano","owner":"Valeria"}},
{"Key":"CAR9", "Record":{"colour":"brown","make":"Holden","model":"Barina","owner":"Shotaro"}}]
~~~
이 프로그램을 자세히 살펴보자. 편집기(예: 원자 또는 시각 스튜디오)를 사용하고 쿼리.js를 여십시오.

애플리케이션은 패브릭 네트워크 모듈에서 두 가지 핵심 클래스인 FileSystemWallet 및 Gateway를 가져오는 것으로 시작한다. 다음 클래스는 지갑에서 사용자1 ID를 찾아 네트워크에 연결하는 데 사용된다.

~~~
const { FileSystemWallet, Gateway } = require('fabric-network');
~~~

애플리케이션은 게이트웨이를 사용하여 네트워크에 연결:
~~~
const gateway = new Gateway();
await gateway.connect(ccp, { wallet, identity: 'user1' });
~~~

이 코드는 새 게이트웨이를 생성한 다음 응용 프로그램을 네트워크에 연결하는 데 사용한다. ccp는 게이트웨이가 지갑에서 ID user1로 액세스하는 네트워크를 설명한다. ccp가 ...../basic-network/connection.json에서 로드되어 JSON 파일로 구문 분석된 방법을 확인하십시오.

~~~
const ccpPath = path.resolve(__dirname, '..', '..', 'basic-network', 'connection.json');
const ccpJSON = fs.readFileSync(ccpPath, 'utf8');
const ccp = JSON.parse(ccpJSON);
~~~

연결 프로필의 구조와 네트워크 정의 방법에 대해 자세히 알아보려면 연결 프로필 항목을 확인하십시오.
네트워크는 여러 채널로 나눌 수 있으며, 다음 중요한 코드 라인은 애플리케이션을 네트워크 내의 특정 채널에 연결한다.

~~~
const network = await gateway.getNetwork('mychannel');
~~~
이 채널 내에서, 우리는 원장과 상호작용하기 위해 스마트 계약 팹카에 접근할 수 있다.

~~~
const contract = network.getContract('fabcar');
~~~
팹카 내에는 많은 다른 거래가 있으며, 우리의 애플리케이션은 처음에 queryAllCars 트랜잭션을 사용하여 원장의 세계 상태 데이터에 액세스한다.

~~~
const result = await contract.evaluateTransaction('queryAllCars');
~~~

평가트랜잭션 방법은 블록체인 네트워크에서 스마트 계약과 가장 간단한 상호 작용 중 하나를 나타낸다. 연결 프로필에 정의된 피어를 선택하고 요청을 해당 요청으로 보내 평가한다. 스마트 계약은 원장의 피어의 사본에 있는 모든 자동차를 조회하여 그 결과를 신청서에 반환한다. 이 상호 작용은 원장을 업데이트하는 결과를 낳지 않는다.

# FabCar 지능형 계약
여기서 내부 트랜잭션을 살펴보겠습니다.FabCar교묘한 계약 탐색chaincode/fabcar/javascript/lib의 근간에 있는 하위 디렉토리fabric-samples열린fabcar.js당신의 편집장에게.

NAT의 스마트 계약이 어떻게 정의되는지 알아보십시오.Contract클래스:
~~~
class FabCar extends Contract {... 
~~~
이 클래스 구조 내에서 다음 트랜잭션이 정의되어 있음을 알 수 있습니다.initLedger,queryCar,queryAllCars,createCar그리고.changeCarOwner. 예:

~~~
async queryCar(ctx, carNumber) {...} async queryAllCars(ctx) {...} 
~~~
좀 더 자세히 살펴보자queryAllCars그것이 원장과 어떻게 상호 작용하는지 확인하기 위한 거래.

~~~
async queryAllCars(ctx) {    const startKey = 'CAR0';   
const endKey = 'CAR999';    
const iterator = await ctx.stub.getStateByRange(startKey, endKey); 
~~~
이 코드는 다음과 같은 차량의 범위를 정의합니다.queryAllCars원장에서 조회합니다. 사이의 모든 차CAR0그리고.CAR999-모든 키에 적절한 태그가 지정되었다고 가정할 때 총 1,000대의 차량이 쿼리를 통해 반환됩니다. 코드의 나머지 부분은 쿼리 결과를 통해 반복되며 애플리케이션에 대해 JSON으로 패키징 합니다.

아래는 응용 프로그램이 스마트 계약에서 다양한 트랜잭션을 호출하는 방법을 보여 줍니다. 각 트랜잭션은 다음과 같은 다양한 API를 사용합니다.getStateByRange원장과 상호 작용하다 이러한 API에 대한 자세한 내용을 볼 수 있습니다

우리는 우리의 것을 볼 수 있다queryAllCars트랜잭션 및 다른 호출createCar우리는 이것을 나중에 튜토리얼에서 사용하여 원장을 갱신하고 블록 체인에 새로운 블록을 추가할 것이다.

하지만 먼저query프로그램 및 변경evaluateTransaction조회 요청CAR4.query프로그램은 다음과 같이 표시되어야 합니다.

~~~
const result = await contract.evaluateTransaction('queryCar', 'CAR4'); 
~~~
프로그램을 저장하고fabcar/javascript안내 책자 자query다시 프로그램:

~~~
node query.js 
~~~

다음 항목이 표시되어야 합니다.

~~~
Wallet path: ...fabric-samples/fabcar/javascript/wallet Transaction has been evaluated, result is: {"colour":"black","make":"Tesla","model":"S","owner":"Adriana"} 
~~~

과거로 돌아가서 거래가 성사된 이후의 결과를 본다면queryAllCars보시다시피CAR4애드리아나의 검은 색 테슬라 모델 S였는데, 여기서 반환된 결과입니다.

우리는 사용할 수 있다queryCar키를 사용하여 차량에 대해 질의하는 트랜잭션입니다(예:CAR0그리고 그 차에 상응하는 모든 제품, 모델, 색상, 소유자를 구하세요.

잘 됐네요. 이 시점에서 당신은 스마트 계약의 기본 쿼리 트랜잭션과 쿼리 프로그램의 몇 안 되는 파라미터에 익숙해져야 해요.

원장 업데이트 시간...

# 원장 업데이트
이제 몇가지 원장 조회와 코드 추가를 완료하였으므로 원장을 갱신할 수 있습니다. 우리가 만들 수 있는 잠재적인 업데이트들이 많이 있습니다. 하지만 새 차를 만드는 것부터 시작해 보죠.

신청서 관점에서 원장의 갱신은 간단하다. 응용 프로그램은 블록 체인 네트워크에 트랜잭션을 제출하고, 트랜잭션의 유효성을 확인하고 커밋 하면 응용 프로그램은 트랜잭션이 성공했다는 통지를 받습니다. 이는 블록 체인 네트워크의 다양한 구성 요소가 함께 작동하여 원장에 대해 제안된 모든 업데이트가 유효하고 일관된 순서로 수행되도록 하는 합의 프로세스를 수반합니다.

위에서 이 프로세스를 작동시키는 주요 구성 요소를 볼 수 있습니다. 각 피어가 원장의 사본을 호스트 하는 여러 피어와 선택적으로 스마트 계약의 사본을 포함하는 네트워크도 주문 서비스를 포함합니다. 순서 지정 서비스는 네트워크의 트랜잭션을 조정하며, 네트워크에 연결된 모든 다른 응용 프로그램에서 시작되는 잘 정의된 순서로 트랜잭션이 포함된 블록을 생성합니다.

우리가 처음으로 원장을 업데이트하면 새 차가 생길 것이다. 우리는 별도의 프로그램을 가지고 있다.invoke.js원장 업데이트에 사용할 것입니다 쿼리와 마찬가지로 편집기를 사용하여 프로그램을 열고 트랜잭션을 구성하고 네트워크에 제출하는 코드 블록으로 이동합니다.

~~~
await contract.submitTransaction('createCar', 'CAR12', 'Honda', 'Accord', 'Black', 'Tom'); 
~~~
응용 프로그램에서 스마트 계약 트랜잭션을 호출하는 방법 보기createCar톰이라는 이름의 소유자와 검은 색 혼다 어코드를 만들기 위해 우리는 사용한다CAR12순차적 키를 사용할 필요가 없다는 것을 보여 주기 위해 식별 키를 사용합니다.

저장하고 프로그램을 실행합니다.


~~~
node invoke.js 
~~~
호출에 성공하면 다음과 같은 출력이 표시됩니다.
~~~
Wallet path: ...fabric-samples/fabcar/javascript/wallet 2018-12-11T14:11:40.935Z - info: [TransactionEventHandler]: _strategySuccess: strategy success for transaction "9076cd4279a71ecf99665aed0ed3590a25bba040fa6b4dd6d010f42bb26ff5d1" Transaction has been submitted 
~~~

주목하세요invoke응용 프로그램이 블록 체인 네트워크와 상호 작용합니다.submitTransactionAPI, 대신evaluateTransaction.

~~~
await contract.submitTransaction('createCar', 'CAR12', 'Honda', 'Accord', 'Black', 'Tom'); 
~~~

submitTransaction보다 훨씬 더 정교하다evaluateTransactionSDK는 단일 피어로 상호 작용하는 대신에submitTransaction블록 체인 네트워크의 모든 필수 조직의 피어에 제안 각 피어는 서명하고 SDK로 반환하는 트랜잭션 응답을 생성하기 위해 이 제안을 사용하여 요청된 스마트 계약을 실행합니다. SDK는 서명된 모든 트랜잭션 응답을 단일 트랜잭션으로 수집한 다음 이를 주문자에게 전송합니다. 주문자는 모든 응용 프로그램에서 트랜잭션을 수집하고 순차적으로 처리합니다. 그런 다음 이러한 블록을 네트워크의 모든 피어에 배포합니다. 여기서 모든 트랜잭션이 검증되고 커밋 됩니다. 마지막으로 SDK에 알림이 표시되어 애플리케이션에 대한 제어 권한을 반환할 수 있습니다.

~~~
메모

submitTransaction또한 트랜잭션의 유효성이 확인되고 해당 트랜잭션이 원장에 커밋 되었는지 확인하는 수신기를 포함합니다. 애플리케이션은 수신기 커밋을 활용하거나 API를 다음과 같이 활용해야 합니다.submitTransaction당신을 위해서라면 이렇게 하지 않았다면, 거래가 성공적으로 진행되지 않았을 수도 있고, 유효성이 확인되지 않았으며, 원장에게 위임되었을 수도 있습니다.
~~~

submitTransaction이 모든 것을 응용 프로그램을 위해! 전산망 상에서 원장의 일관성을 유지하기 위해 응용 프로그램, 스마트 계약, 동료 및 주문 서비스가 함께 작용하는 과정을 합의라고 하며, 이를 자세히 설명한다.

이 거래가 원장에게 작성되었는지 확인하려면,query.js그리고 논쟁을 바꾸다CAR4로.CAR12.

즉, 다음을 변경합니다.
~~~
const result = await contract.evaluateTransaction('queryCar', 'CAR4'); 
~~~
여기까지:
~~~
const result = await contract.evaluateTransaction('queryCar', 'CAR12'); 
~~~
한번 더 저장한 다음 쿼리:

~~~
node query.js 
~~~
이렇게 하면 다음이 반환됩니다.

~~~
Wallet path: ...fabric-samples/fabcar/javascript/wallet Transaction has been evaluated, result is: {"colour":"Black","make":"Honda","model":"Accord","owner":"Tom"} 
~~~
축하해요. 당신은 차를 만들어서 그것이 원장에 기록된 것을 확인했습니다!

이제 그 일을 끝냈으니, 톰이 관대하다고 생각하고 데이브라는 사람에게 혼다 어코드를 주고 싶어 한다고 가정해 보자.

이렇게 하려면 다음으로 돌아가십시오.invoke.js스마트 계약 트랜잭션 변경createCar로.changeCarOwner입력 인수의 해당 변경 사항:
~~~
await contract.submitTransaction('changeCarOwner', 'CAR12', 'Dave'); 
~~~
첫번째 인수-CAR12—소유자를 변경할 차량을 식별합니다. 두번째 인수-Dave— 새로운 자동차 소유자를 정의합니다.

프로그램을 저장하고 다시 실행합니다.
~~~
node invoke.js 
~~~
이제 다시 대장을 조회하여 데이브가 현재CAR12키:

~~~
node query.js 
~~~
다음 결과를 반환해야 합니다.

~~~
Wallet path: ...fabric-samples/fabcar/javascript/wallet Transaction has been evaluated, result is: {"colour":"Black","make":"Honda","model":"Accord","owner":"Dave"} 
~~~
의 소유권CAR12톰에서 데이브로 바뀌었다.

~~~
메모

실제 애플리케이션에서는 스마트 계약이 접속 제어 논리를 가지고 있을 가능성이 있다. 예를 들어, 허가 받은 특정 사용자만 새 차를 만들 수 있으며, 자동차 소유자만 다른 사람에게 차를 옮길 수 있습니다.
~~~

# 정리
이제 몇 가지 쿼리 및 몇 가지 업데이트를 수행했으므로, 당신은 애플리케이션이 스마트 계약을 사용하여 블록체인 네트워크와 어떻게 상호 작용하는지 상당히 잘 알고 있어야 한다. 스마트 계약, API 및 SDK가 쿼리 및 업데이트에서 수행하는 역할의 기본을 살펴보았으며, 다른 종류의 애플리케이션이 다른 비즈니스 작업 및 운영을 수행하는 데 어떻게 사용될 수 있는지에 대한 느낌이 있어야 한다.

# 추가 자원
서론에서 말했듯이, 우리는 스마트 계약, 프로세스 및 데이터 설계에 대한 심층적인 정보, 보다 심층적인 상업용 종이 튜토리얼을 사용하는 튜토리얼 및 애플리케이션 개발과 관련된 많은 양의 기타 자료를 포함하는 애플리케이션 개발에 관한 전체 섹션을 가지고 있다.