# 오브스(Orbs) 블록체인 노드 설치하기 (Polygon CLI 사용)


이 가이드 문서에서는 새로운 노드를 생성하여 오브스 네트워크에 연결하는 방법을 순서대로 안내하고 있습니다.

![](../diagram.png)

## 미리 준비할 것들

가이드대로 하려면 아래 사항들을 먼저 확인하고 준비해야합니다:

- PC - 맥(Mac) 또는 리눅스 (본 가이드는 Mac을 기준으로 작성하였으므로 가능하면 Mac 환경 권장)
- 이더리움 Endpoint URL. 에서는 [Infura 무료 계정](infura-setup-free.md)을 이용할 수 있습니다.
- **새로운 AWS 계정**
  - V1 노드가 운영중이라면 같은 AWS 계정을 사용해도 괜찮습니다
- AWS client 설치 - `brew install awscli` 명령을 통해 PC에 설치할 수 있습니다

  awscli는 AWS에 원격으로 접속할 수 있는 클라이언트 프로그램입니다.

- 노드에서 사용한 새로운 주소. [아래 내용](#노드용-주소와-프라이빗-키Private-Key-준비)을 참조하세요.
- 가디언 등록시 사용할 가디언 지갑 (Windows에서 진행가능)
- 메타마스크 지갑 (Windows에서 진행가능)
- AWS credentials 프로파일 설정:

  `aws configure` 명령어로 기본 "default" 프로파일을 설정할 수 있습니다. aws 프로파일 설정에 대한 자세한 내용은 [aws 문서](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html)를 참조하세요.

  AWS 계정에 대한 관리자 권한은 위에서 입력한 프로파일에 구성해야합니다.

- [Node.js](https://nodejs.org/en/) version 8 또는 그 이상 버전 설치
  
  `brew install node` 명령어로 Node.js를 설치하세요

- [Terraform](https://www.terraform.io/downloads.html)
  
  v0.12.23 버전의 Terraform을 설치해야 합니다.

  아래의 설치 명령을 순차적으로 실행하세요.
  
  `brew install tfenv`
  
  `tfenv install 0.12.23`
  
  `tfenv use 0.12.23`
  
  설치후, `terraform -v` 명령으로 설치된 버전을 확인해보세요. `Terraform v0.12.23` 이 표시되면 정상입니다.

### SSH public and private keys 생성하기

기존 노드 인스턴스에 ssh public key를 사용하고 있다면, 이 부분은 건너뛰어도 됩니다.

스크립트에서 AWS의 EC2 사용하여 설치를 진행하기 위해서는 public/private 키조합이 필요합니다. 이 키 파일은 설치 중 설정값으로 사용될 때를 제외하고는 사용할 수 없도록 안전하게 보관해야합니다 (아래 `orbs-node.json`파일을 작성할때 pub 파일의 위치를 입력해야합니다)

생성된 키값은 별도의 암호문(passphrase)설정을 __해서는 안됩니다__.

[GitHub](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)에서 안내하는 튜토리얼을 따라하시거나 그 외 가능한 다른 방법으로 키를 생성하시면 됩니다.

간단한 방법을 알려드리자면 다음과 같은 명령어 실행으로 키를 생성할 수 있습니다:

    ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
    
* -C "your_email@example.com" 은 옵션으로 입력하지 않아도 무방합니다

### AWS에서 고정 IP 할당받기

Orbs 노드는 고정 IP (Elastic IP)를 사용하여 네트워크에서 통신하게 됩니다.

- AWS Console에 로그인하세요
- 노드를 설치할 지역을 선택하세요 (예: `ca-central-1`)
- Elastic IP 하나를 신청하여 할당받으세요

노드 IP 주소와 선택지역은 아래 노드 설정 파일에 입력시 필요합니다.
IP 주소는 또한 [가디언 등록하기](#가디언-등록하기)에서 입력값으로 필요합니다.

### 노드용 주소와 프라이빗 키(Private Key) 준비

Orbs 노드 주소는 표준 이더리움 주소를 사용합니다. 이 주소는 (1) Orbs 네트워크에서 블록체 서명할 때, (2) 이더리움 상에서 PoS 스마트 컨트랙트에 트랜잭션을 내보낼 때 사용하게 됩니다. 그러므로, 해당 노드는 주소에 대한 프라이빗 키(private key)를 가지고 있어야 합니다. 이 주소와 프라이빗 키는 가디언 주소와는 다른 주소와 프라이빗 키이어야 합니다.

일상적으로 노드가 운영되는 동안, 자동으로 노드는 이더리움상에 있는 PoS 스마트 컨트랙트에 트랜잭션(리워드 배포 실행이나 위원회에 등록하기 위한 준비작업, 네트워크 동기화 신호등)을 내보냅니다. Orbs 노드의 주소에는 이러한 PoS 컨트랙트 실행용 트랜잭션을 위한 가스비용이 충분히 들어있어야 합니다. 그러므로 각 가디언은 주기적으로 자신의 노드 주소에 적어도 0.5 ETH 이상이 들어있는지 항상 확인해야만 합니다.

중요 - 최초 등록과정에서는 최소 1 ETH가 노드 주소에 들어있어야 합니다.

노드 주소와 프라이빗 키를 얻을 때 및 보관시 보안에 유의하여주시고, 프라이빗 키는 아래 노드 등록과정에서 필요합니다.

노드 주소 또한 [가디언 등록하기](#가디언-등록하기)에서 입력값으로 필요합니다.

노드 주소는 추후 변경이 가능합니다. 주소를 변경할 경우에는 반드시 기존 노드를 정지시킨 후, 변경된 주소를 새로 설치해야합니다.

프라이빗 키는 반드시 안전한 곳에 보관해야 합니다.

프라이빗 키를 확인하는 여러가지 방법이 있지만, 그 중 MyCrypto 지갑에서 확인하는 방법은 [여기](https://github.com/orbs-network/validator-instructions/blob/master/public/key_generation.md)를 참조해주세요.

### Polygon 설치하기 (npm 이용)

다음 명령어를 입력하여 Polygon을 설치하세요

    npm install -g @orbs-network/polygon

이미 예전에 Polygon을 설치하였다면 새로 노드를 설치하기 전에 다음 명령어를 통해 최신 버전으로 업데이트해주세요 

    npm update -g @orbs-network/polygon


### 노드 설치를 위한 폴더 만들기

다른 파일들과 섞이지 않도록 노드 설정을 저장할 폴더를 따로 만들어주세요(폴더명 예시: "orbs-v2"). 이 폴더에는 설정파일과 로그, 향후 노드 제거시 필요한 데이터가 기록됩니다.

    mkdir orbs-v2

__이 폴더는 설치가 완료되더라도 삭제하면 안되며, 다른 곳에 복사해서 백업해 두십시오.__

새로 만든 폴더로 이동하여 아래 순서를 계속 진행해주세요.

    cd orbs-v2

### 설치를 위한 JSON 파일 생성, 설정하기

폴더 안에 `orbs-node.json` 파일을 새로 만들어주세요.

`orbs-node.json`파일의 내용은 다음과 같습니다:

    {
        "name": "노드이름(영문으로)",
        "awsProfile": "AWS 프로파일",
        "sshPublicKey": "ssh 퍼블릭 키 파일 위치",
        "orbsAddress": "노드용 이더리움 주소",
        "region": "AWS 지역",
        "publicIp": "노드 IP 주소",
        "nodeSize": "r5.large",
        "nodeCount": 0,
        "cachePath": "./_terraform",
        "incomingSshCidrBlocks": ["ssh 접근제한 설정",...],
        "managementConfig": {
            "orchestrator": {
                "DynamicManagementConfig": {
                    "Url": "http://localhost:7666/node/management",
                    "ReadInterval": "1m",
                    "ResetTimeout": "30m"
                },
                "storage-driver": "local",
                "storage-mount-type": "bind"
            },
            "services": {
                "management-service": {
                    "InternalPort": 8080,
                    "ExternalPort": 7666,
                    "DockerConfig": {
                        "Image": "orbsnetwork/management-service",
                        "Tag": "bootstrap",
                        "Pull": true
                    },
                    "Config": {
                        "BootstrapMode": true,
                        "EthereumEndpoint": "이더리움 endpoint url",
                        "DockerNamespace":"orbsnetwork"
                    }
                }
            }
        }
    }

위 내용에서 수정입력할 값은 다음과 같습니다:
* `노드이름(영문으로)` - AWS에서의 리소스명 입력 규칙을 따라 자신의 노드명을 입력해주세요. 이 이름은 가디언이름이 아니며 네트워크에서 노드 리소스 구분을 위해 사용됩니다. S3 버킷명 형식을 따라주세요 (소문자와 빼기기호 사용가능. 사용불가:대문자,특수기호나 빈칸, 언더바(_)를 사용할 수 없음.)
* `AWS 프로파일` - AWS 계정 연결을 위해 설정한 "default" 또는 직접 설정한 AWS 프로파일 명을 입력해주세요. AWS 계정이 여러개라면 알맞은 계정에 설정된 프로파일을 입력해야합니다.
* `ssh 퍼블릭 키 파일 위치` - SSH 퍼블릭 key 파일 경로 (따로 변경하지 않았다면 기본값은 `~/.ssh/id_rsa.pub`일 것입니다)
* `노드용 이더리움 주소` - 노드용 이더리움 주소값 중 __맨 앞의 '0x'를 제외한 값__ 을 입력합니다
* `AWS 지역` - AWS 지역값. 설치하는 노드는 이 지역에 프로비저닝됩니다. (예: `ca-central-1`)
* `노드 IP 주소` - 노드에 할당될 고정 IP주소입니다. 선택된 AWS지역에서 신청할당받은 Elastic IP를 입력해야합니다.
* `이더리움 endpoint url` - infura에서 받은 이더리움 노드 URL입니다. 
* `ssh 접근제한 설정` - 하나이상의 CIDR 설정을 통해 노드로의 ssh접근을 제한할 수 있습니다. ssh 퍼블릭 키를 통해 접근하므로 평소에는 제한하지 않아도 괜찮지만, 문제가 있는 경우 제한해야할 필요가 있을 수 있습니다. 형식은 표준 CIDR을 따릅니다. 설정되지 않은 IP는 SSH 키가 있어도 접근이 허용되지 않습니다. 별다른 IP접근 제한을 하지 않으려면, `"incomingSshCidrBlocks": ["0.0.0.0/0"],` 를 입력할 수 있습니다.

### Polygon CLI를 이용하여 노드 설치하기

임시로 `privatekey`라는 파일을 생성하여 프라이빗 키를 저장해주세요. 이때 __맨 앞의 0x__ 는 제외하고 입력합니다. 참고로, 프라이빗 키는 64글자의 숫자와 알파벳문자열로 이루어져 있습니다. (예: `f5f83Ee70a85fFF2exxxxxxxxxxxxxxxxxxxxxxxxxxx334932F34C8D629165Ed`)

준비가 다 되었으면 아래 명령어로 노드 설치를 시작합니다:
(위에서 진행시 파일명을 다르게 만들었다면 아래 명령어에서 해당 파일명에 맞게 수정입력해주세요)

    polygon create -f orbs-node.json --orbs-private-key $(cat privatekey)

프라이빗 키는 노출되지 않도록 안전한 곳에 별도로 보관해주시고, 위에서 임시로 사용한 `privatekey` 파일은 설치가 완료되면 삭제해주세요.

노드를 설치하는 동안 Terraform의 캐시파일이 json내 설정된 `cachePath` 경로에 저장됩니다. 이 데이터는 노드 설치가 완료된 후 계속 유지되어야 하며 가능하면 백업해 둘 것을 권장합니다. 추후 노드 삭제등에 필요합니다.

필요시, 노드를 제거하고 모든 aws 리소스를 해제하는 명령어는 다음과 같습니다:
         
    polygon destroy -f orbs-node.json
    
### 꼭 확인해야할 중요 내용! ###

설치가 완료 된 후, 반드시 설치에 사용한 폴더는 안전한 곳에 백업해 두세요:
After deployment make sure to backup and securely store the deployment folder data, including:

* __키 값들__ (프라이빗 키, AWS 계정 credential 관련 키 값, SSH 키값등)

* _terraform 캐시 폴더 - 향후 노드 제거 및 재설치시 필요

* `orbs-node.json` 파일

특히 프라이빗 키와 같은 데이터는 일반 폴더등 안전하지 않은 곳에 남겨두면 안됩니다.

### 오브스 노드 설정 업데이트

__노드 설정을 변경하는 경우에는 반드시 노드가 제거되고 내려간 상태에서 진행되어야 합니다.__

노드 설정을 업데이트하는 방법

1. polygon destroy 명령어 노드 제거 실행 `polygon destroy -f orbs-node.json`
2. JSON 파일 수정 (수정할 항목 변경)
3. polygon create 명령어로 다시 설치실행 ([설치항목](#Polygon-CLI를-이용하여-노드-설치하기))

### 가디언 등록하기

네트워크에 가디언을 등록하려면 별도의 등록용 웹사이트앱에 접속해야합니다. 아래 링크를 통해 해당 웹에서 안내를 따라 등록을 완료해주세요. 메타마스크를 통한 지갑 연동이 필요합니다.

[가디언 등록하러 가기](https://guardians.orbs.network/registration) (메타마스크 지갑 필요)

### 노드 설치가 올바르게 완료되었는지 확인

모든 서비스가 문제없이 노드에서 실행되고 있는지 확인해보세요. 브라우저에서 아래 주소를 입력해보세요. ( `<node ip>`는 노드 등록에 사용한 Elastic IP 주소입니다)
```
http://<node ip>/services/boyar/status
```

PoS 네트워크에서의 노드 상태를 아래 주소에서 확인할 수 있습니다 (가디언 등록까지 모두 완료하고 10분 후에 확인해보세요)
```
http://<node ip>/services/management-service/status
```

__모든 설정이 완료되었습니다. 축하드립니다!__

## 문제 발생시 요령



1. __IP주소를 발견할 수 없다는 에러발생 시__: AWS에서 어느 지역(Region)인지 다시 확인해보세요. json 파일에 설정한 지역에서 할당받은 Elastic IP가 맞는지 확인해보세요.

2. __확인용 주소가 브라우저에서 제대로 응답하지 않는 경우__: 이더리움 노드 싱크가 완료되지 않았을 수 있습니다. 몇 시간 후에 다시 시도해보세요.

3. __Terraform 관련 에러가 있는 경우__: 작업중인 PC의 시간이 실제 시간과 어긋나 있지 않은지 확인해보세요.

4. 그 외 문제가 있을시, 오브스 팀에 문의해주세요.
