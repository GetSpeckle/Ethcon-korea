# 로컬 환경에서 서브스트레이트 사용하기

## 설치

### 맥이나 우분투 환경에서

bash 터미널을 연 뒤, 아래 커맨드들을 입력해주세요.

```bash
curl https://sh.rustup.rs -sSf | sh

rustup update nightly
rustup target add wasm32-unknown-unknown --toolchain nightly
rustup update stable
cargo install --git https://github.com/alexcrichton/wasm-gc
```

또한 아래와 같은 패키지들을 설치해줍니다.

- 리눅스:

```bash
sudo apt install cmake pkg-config libssl-dev git clang libclang-dev
```

- 맥:

```bash
brew install cmake pkg-config openssl git llvm
```

이후 설치를 끝내기 위해 [공통과정](#공통과정)으로 이동합니다.

### 윈도우 환경에서

1. "Build Tools for Visual Studio"를 설치해주세요.

    - 다운로드 링크: [https://aka.ms/buildtools](https://aka.ms/buildtools)
    - 설치 파일을 실행: `vs_buildtools.exe`
    - Visual C++ 빌드 툴을 설치할 때는 Windows 10 SDK 설치 항목이 체크되어 있는 지 확인해주세요.
  ![윈도우 빌드 툴 설치](https://camo.githubusercontent.com/7746a4f8f8658e553bc74a204c115827a173c34f/68747470733a2f2f692e696d6775722e636f6d2f7a6179564c6d752e706e67)
    - 컴퓨터를 재시작해주세요

2. 러스트를 설치해야합니다.

    - 러스트 설치에 관한 자세한 안내사항은 [Rust Book 한국어 번역본 by rinthel](https://rinthel.github.io/rust-lang-book-ko/)에 있습니다.
    - 다운로드 링크: [https://www.rust-lang.org/tools/install](https://www.rust-lang.org/tools/install)
    - `rustup-init.exe` 설치파일을 실행시켜 주세요. (참고: 이 파일이 실행되면 전에 빌드 툴 깔 때 vs_buildtool을 깔라고 창이 뜨지 않아야 합니다.)
    - 기본 옵션으로 설치를 하세요(그저 `다음`만 클릭하면 됩니다.)
    - 러스트를 실행을 하려면 Cargo의 바이너리 실행파일 폴더(%USERPROFILE%\.cargo\bin)이 윈도우 `PATH` 환경 변수에 추가가 되어야 합니다. 그리고 커맨드 프롬프트를 다시 시작해야 할거에요.
    
3. 그러고 나면, 커맨드 프롬프트에서 아래 커맨드들을 입력하여 웹어셈블리 빌드 환경을 설정해야 합니다:

```powershell
rustup update nightly
rustup update stable
rustup target add wasm32-unknown-unknown --toolchain nightly
```

4. 다음으로, wasm 파일의 용량을 줄이기 위한 wasm-gc(가비지 컬렉터)를 설치해줍니다:

```powershell
cargo install --git https://github.com/alexcrichton/wasm-gc --force
```

5. 그러고 나면, LLVM을 설치해야 합니다. 링크: [https://releases.llvm.org/download.html](https://releases.llvm.org/download.html)

6. 또 안전한 연결을 위해 OpenSSL을 `vcpkg`를 이용하여 설치합니다:

```powershell
mkdir \Tools
cd \Tools
git clone https://github.com/Microsoft/vcpkg.git
cd vcpkg
.\bootstrap-vcpkg.bat
.\vcpkg.exe install openssl:x64-windows-static
```

7. 그게 끝나면 이제 OpenSSL을 시스템 환경변수에 더해줍니다:

```powershell
$env:OPENSSL_DIR = 'C:\Tools\vcpkg\installed\x64-windows-static'
$env:OPENSSL_STATIC = 'Yes'
[System.Environment]::SetEnvironmentVariable('OPENSSL_DIR', $env:OPENSSL_DIR, [System.EnvironmentVariableTarget]::User)
[System.Environment]::SetEnvironmentVariable('OPENSSL_STATIC', $env:OPENSSL_STATIC, [System.EnvironmentVariableTarget]::User)
```

8. 그리고 정말 마지막으로 cmake를 설치합니다. 링크: [ https://cmake.org/download/]( https://cmake.org/download/)


### 공통과정

서브스트레이트 코드를 다운받으세요.

```bash
git clone https://github.com/paritytech/substrate.git
cd substrate
```

그리고 빌드시키세요.

```bash
./scripts/build.sh          # Builds the WebAssembly binaries
cargo build                 # Builds all native code
```

테스트도 한번 돌려보셔도 됩니다.

```bash
cargo test --all
```

아니면 한 패키지에 대한 테스트만 돌려봐도 됩니다(i.e. `cargo test -p srml-assets`)


개발용으로 체인을 돌리고 싶으면

```bash
cargo run \-- --dev
```

디테일한 로드는 다음과 같은 환경변수를 설정하면 받을 수 있습니다:
`RUST_LOG=debug RUST_BACKTRACE=1 cargo run -- --dev`

만약 여러개의 노드가 컨센서스 알고리듬을 처리하는 걸 로컬 환경에서 보고 싶다면, 처음에 validator로 설정한 Alice와 Bob을 위한 로컬 테스트넷을 만들면 됩니다.
genesis snapshot에서 테스트넷 DOT을 부여받은 상태로 말이지요. 각 블록체인 노드마다 이름을 부여한 뒤 [Telemetry](https://telemetry.polkadot.io/#/Local%20Testnet)에 올려봅시다.
이를 위해서는 두 개의 터미널이 필요한데요.

일단 Alice꺼부터 만들어보죠. 일단 앨리스의 부트노드 아이디(Bootnode ID)는 `--node-key` 옵션값에서 나온 `QmQZ8TjTqeDj3ciwr93EJ95hxfDsb9pEYDizUAbWpigtQN`입니다:

```bash
cargo run --release \-- \
  --base-path /tmp/alice \
  --chain=local \
  --alice \
  --node-key 0000000000000000000000000000000000000000000000000000000000000001 \
  --telemetry-url ws://telemetry.polkadot.io:1024 \
  --validator
``` 

두 번째 터미널에서는 Bob꺼를 만들어보죠. tcp 포트는 로컬에서 다른 포트를 사용하게 30334로 맞춰주고, bob의 블록체인 데이터를 저장한 디렉토리인 `/tmp/bob`을 설정해줍니다.
`--bootnodes` 옵션을 설정해서 앨리스의 tcp 포트 30333을 사용하는 노드에 연결시켜줍니다:

```bash
cargo run --release \-- \
  --base-path /tmp/bob \
  --bootnodes /ip4/127.0.0.1/tcp/30333/p2p/QmQZ8TjTqeDj3ciwr93EJ95hxfDsb9pEYDizUAbWpigtQN \
  --chain=local \
  --bob \
  --port 30334 \
  --telemetry-url ws://telemetry.polkadot.io:1024 \
  --validator
```

다른 서브스트레이트 옵션(e.g. rpc-port 설정, tcp 포트 설정 등)은 

배포된 버전은 `substrate --help`이고

개발 버전은 개발하고 있는 git 프로젝트 폴더에서 `cargo run -- --help`를 입력하면 됩니다.


## 서브스트레이트 돌려보기

### 맥이나 우분투 환경에서

터미널을 연 뒤 아래 코드를 복붙하고 입력시키세요:

```bash
curl https://getsubstrate.io -sSf | bash
```

서브스트레이트를 로컬환경에서 실행시키려면 `substrate --dev`를 터미널에 입력하면 됩니다.

이제 나만의 블록체인 네트워크나 암호화폐를 만드려면 서브스트레이트의 체인 설정을 담당하는 
"chainspec"파일을 변경시켜줘야 합니다.

일단 맞춤 설정을 위한 chainspec 템플릿을 얻기 위해 아래 bash 명령어를 입력해줍니다.
참고로 저희는 기본적인 의도로 설정되는 타입인 "staging" 체인을 이용하여 만듭니다.

```bash
substrate build-spec --chain=staging > ~/chainspec.json
```

자 이제 chainspec 파일의 내용을 상황에 맞게 바꿔봅시다. 물론 블록생성 주기 외에도 다른 속성값들을 조정할 수 있지만,
현재는 가장 알아보기 쉬운 것부터 해보자고요.

`chainspec.json`에서 "timestamp"라는 키를 찾은 뒤 아래와 같이 내용을 바꿔봅시다.

```json
      "timestamp": {
        "period": 10
      },
```

자, 이제 내용을 수정했으면 아래와 같은 커맨드로 수정된 `chainspec.json` 파일을 이용하여 "가공되지 않은" 블록체인 견적을 뽑아낼 수 있습니다.

```bash
substrate build-spec --chain ~/chainspec.json --raw > ~/mychain.json
```


이제 만들어진 견적 파일을 서브스트레이트에 입력시켜줍니다.

```bash
substrate --chain ~/mychain.json
```

이 커맨드는 블록 생성을 하지 않아서 딱히 큰 결과를 보여주지는 않지만 폹카닷 네트워크에서의 validator로 블록체인을 작동시키는 `--validator` 옵션과 `--key` 옵션으로 처음에 블록을 검증할 공유키를 설정해줌으로서 배포할 블록체인을 만들어줍니다.

```bash
substrate --chain ./mychain.json --validator --key ...
```

`mychain.json`파일(미리 설정된 공유키들과 함께)를 공유해서 위와 같은 방법을 알려준 뒤 각자 설정한 블록체인을 공유해보세요! 🚀🚀🚀

