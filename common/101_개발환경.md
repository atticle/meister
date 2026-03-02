# Windows 컴퓨터에 파이썬 개발환경 구축
- 윈도우 운영체제 기준으로 시스템 의존성과 저장소 낭비를 최소화해 설치.
- 파이썬(마이크로파이썬 포함) 기반 데이터과학, IoT, 인공지능, 로봇(ROS2), 자동제어 분야에서 사용

## 윈도우 CLI 명령 (cmd, pwsh)
> 사용자가 직접 파이썬 IoT 개발환경을 구축하려면 CLI 명령에 익숙해야 함

- CLI(command Line Interface)는 텍스트 환경에서 명령어로 시스템 제어 및 사용자 작업 수행
  - 윈도우 운영체제는 명령 프롬프트(cmd)와 파워쉘(powrshell)을 통해 CLI 환경 지원
    - 최근에 개발되어 좀더 리눅스 환경에 가까운 파워셀 권장
  - CLI 실행
    - 실행창(Win+r)에서 cmd 또는 powershell 입력 후 <enter>
      - win+r은 <윈도우>와 <r>키를 함께 누름
- 운영체제는 저장소에서 파일을 관리할 때 폴더 단위로 묶어 **트리 구조**로 관리
  - 폴더도 파일의 일종으로 자신이 관리하는 파일 또는 하위 폴더 정보를 내용으로 가짐
  ```sh
  C:\VSCode
       |--- data
       |--- ext     
             |--- python
  ```  
- 트리 구조의 최상위 폴더를 루트(\)라 하며, 그 아래 하위 폴더를 가짐
  - 윈도우 운영체제는 드라이버 문자(C:\) 포함
  - 폴더와 폴더 사이는 구문 문자 사용
    - 리눅스는 /, 윈도우는 \(기본값) 또는 /
  - 예약 문자
    - 현재 폴더: .
    - 상위 폴더: ..
    - 홈 폴더: ~
      - 홈 폴더는 로그인 계정에 부여되며, 해당 사용자가 모든 권한을 갖는 폴더
      - 윈도우 운영체제는 C:\User 아래 로그인 이름의 하위 폴더 위치
- 경로(Path)는 트리 구조에서 루트 또는 현재 폴더를 기준으로 작업 폴더까지의 계층 구조(위치)를 나타냄
  - 현재 작업 폴더(또는 현재 폴더)는 사용자가 작업을 수행하기 위해 이동한 폴더
  - 절대 경로는 루트로부터 작업 폴더까지의 경로
     - 예: C:\VSCode\ext\python
  - 상대 경로는 현재(.) 또는 상위(..) 폴더를 기준으로 작업 폴더까지의 경로
     - 예: ..\..\data

- 폴더 이동은 cd 명령 사용
  ```sh
  cd ~
  cd c:\
  ```
- 폴더 내용 확인은 ls 명령 사용
  ```sh
  cd ~
  ls
  ```
- 현재 작업 폴더의 절대 경로 확인은 pwd 명령 사용
  ```sh
  cd ~
  pwd
  ```
- 폴더 생성은 md 명령 사용
  ```sh
  cd c:\
  md VS_Code
  ls  
  ```
- 폴더(파일) 이름 변경은 mv 명령 사용
  ```sh
  cd c:\
  mv VS_Code VSCode
  ls
  ```
- 폴더(파일) 삭제는 rm 명령 사용
  ```sh
  cd c:\
  rm VSCode
  ls
  ```

<details>
<summary>기본 명령 요약</summary>

|command	|Description|
-------|--------------------------------
call	| calls a batch file from another one
**cd**	| **change directory**
**cls / clear**	| **clear screen**
cmd	| start command prompt
color	| change console color
date	| show/set date
**dir / ls**	| **list directory content**
echo	| text output
exit	| exits the command prompt or a batch file
find	| find files
hostname	| display host name
pause	| pauses the execution of a batch file and shows a message
runas	| start a program as another user
shutdown	| shutdown the computer
sort	| sort the screen output
start	| start an own window to execute a program or command
taskkill	| terminate a process or a application
tasklist	| display applications and related tasks
time	| display/edit the system time
timeout	| wait any time
title	| set title for prompt
ver	| display operating system version
w32tm	| setting time synchronisation/time server/time zone

</details>

<details>
<summary>파일 명령 요약</summary>

|command	|Description|
-------|--------------------------------
attrib	|display file attributes|
comp	|compare file contents
compact	|display/change file compression
**copy / cp / xcopy**	|**copy files**
diskcomp	|compare content of two floppy disks
diskcopy	|copy floppy disc to another one
**erase / del / rm**	|**delete one or more files**
expand	|extract files
fc	|copare files and display the differences
**mkdir / md**	|**create a new directory**
**move / mv**	|**move/rename files**
replace	|replace files
**rmdir / rd**	|**delete directory**
**tree**	|**display folder structure graphically**
type	|display content of text files
</details>

<details>
<summary>네트워크 명령 요약</summary>

|command	|Description|
-------|--------------------------------
ftp	|transfer files to a FTP server
ftype	|display file type and mapping
getmac	|display MAC address
**ipconfig**	|**display IP network settings**
netsh	|configure/control/display network components
netstat	|display TCP/IP connections and status
nslookup	|query the DNS
pathping	|test the connection to a specific IP address
**ping**	|**pings the network**
route	|display network routing table, add static routes
systeminfo	|displays computer-specific properties and configurations
telnet	| establish Telnet connection
tftp	|transfer files to a TFTP server
tracert	|trace routes similar to patchping
**ssh**	| **establish SSH connection**
</details>
</details>

## 사용자 설치
### VSCode 포터블 설치
> 별도 설치 프로그램 없이 압축 파일을 다운로드해 사용
1. 윈도우용 최신 버전 다운로드:  https://code.visualstudio.com/docs/?dv=winzip
   - VSCode-win32-x64-x.y.z.zip (x, y, z는 버전 번호)
2. C 드라이버 루트 경로(C:\\)에 압축 해제
   - 바탕화면은 권장하지 않음.
3. 탐색기에서 C:\를 탐색한 후 폴더명을 **VSCode**로 변경 (이하 VSCode 루트)
4. VSCode 루트에 data 폴더 생성 (C:\VSCode\data)
   - data 폴더에는 사용자가 설치한 확장 기능 및 동적 데이터가 저장됨

### 파이썬3 임베디드 SDK 설치
> 별도 설치 프로그램 없이 압축 파일을 다운로드해 사용하며, 파이썬 SDK는 파이썬 인터프리터와 표준 라이브러리로 구성
>> 임베디드 버전은 VSCode와 같은 개발환경을 위해 IDE나 pip, tkinter(GUI 라이브러리) 등은 생략하고 꼭 필요한 핵심 라이브러리만 제공함

1. 윈도우용 V3.x.y 버전 다운로드
   - Windows embeddable package (64-bit) (embed는 Embeddable package의 약자로 꼭 필요한 기능만 제공)
2. VSCode 루트에 data 폴더 생성 (C:\VSCode\data\lib) 
3. data 폴더에 다운받은 파이썬 SDK 압축 해제
4. 탐색기를 이용해 폴더명을 **python**로 변경 (이하 python 루트)
   - C:\VSCode\data\lib\python 
5. 외부 패키지(라이브러리) 경로를 지정하는 python3xy._pth 파일 이름 변경
   - _python3xy._pth 
  
### 파이썬 패키지 관리자(pip) 설치
> 패키지는 라이브러리의 일종으로 모듈로도 불리며, pip는 일반적으로 pypi.org를 통해 배포되는 라이브러리 설치 도구
>> 옵션을 통해 패키지 탐색(search), 다운로드(download), 설치(install), 설치된 패키지 목록 확인(list), 제거(uninstall) 지원

1. 부트스트립 최신 버전 얻기: https://bootstrap.pypa.io/
   - 부트스트립은 실행 파일 안에 실행 환경(운영체제 버전, 실행 경로)을 내장해 더 빠른 실행 보장
   - 시스템 의존성을 줄이려면 부트스트립된 pip.exe보다 pip 모듈 사용 권장 (python -m pip ...)
3. 목록에서 get-pip.py를 우클릭 후 컨텍스트 메뉴가 표시되면 **다른 이름으로 링크 저장** 선택
4. 다른 이름으로 저장 창이 표시되면 python 루트 선택
   - C:\VSCode\data\lib\python\get-pip.py
5. CLI(명령 프롬프트 또는 파워쉘)을 실행한 후 현재 경로를 python 루트로 변경한 후 pip 설치
   ```sh
   cd C:\VSCode\data\lib\python
   python .\get-pip.py
   ```
   - pip 및 wheel 패키지(라이브러리)와 실행 파일이 다운로드됨
     - 패키지는 python 루트의 Lib/site-packages 폴더에 위치
     - 실행 파일은 python 루트의 Scripts 폴더에 위치

### 학습과 관련된 필수 패키지 설치
> 파이썬은 표준 라이브러리 수준의 외부 패키지가 다수 존재함

0. 주의 사항
- 환경변수 path에 python 루트 경로(C:\VSCode\data\lib\python)를 추가할 때까지는 python 루트에서 실행
  ```sh
  cd C:\VSCode\data\lib\python
  ```
- 환경변수 path에 Scripts 경로(C:\VSCode\data\lib\python\Scripts)를 추가할 때까지는 python -m pip로 패키지 설치
  ```sh
  python -m pip install <설치할 패키지 이름>
  ```

1. 데이터 처리 및 시각화
```sh
python -m pip install numpy
python -m pip install matplotlib
python -m pip install pandas
```

2. Jupyter(IPython) 지원
```sh
python -m pip install ipykernel
```

3. PC 마우스/키보드 자동화
```sh
python -m pip install pyautogui
```

4. 영상처리
> cvzone을 설치하면 opencv-python(이미지 프로세싱) 패키지도 함께 설치됨
```sh
python -m pip install mediapipe
python -m pip install cvzone
```

5. 인터넷(IoT) 통신
```sh
python -m pip install paho-mqtt
```

6. 마이크로파이썬 기반 실습장치(XNode)를 위한 툴
> pyqt6, pyside6, pyqtgraph, pyopengl, pyserial-asyncio, click, python-dotenv 패키지도 함께 설치됨
```sh
python -m pip install xnode
```

### python 루트 및 외부 모듈 실행 파일 경로를 환경변수 path에 추가
> CLI에서 현재 경로가 어디든 관계없이 python 또는 pip 명령을 실행하려면 환경변수 path에 해당 프로그램 경로가 등록되어 있어야 함

0. 주의
  - 수정한 환경변수는 응용 프로그램(CLI 환경 포함)을 다시 실행해야 적용됨
  - 환경변수 PATH를 잘못 수정하면 기존 응용프로그램이 실행되지 않을 수 있으므로 반드시 백업한 후 수행할 것
  - 아래 2가지 방법 중 하나 선택

1. CLI  기반
   1. 관리자 권한으로 cmd 실행
   2. 기존 환경변수 PATH 백업
   ```sh
   cd %userprofile%
   echo %PATH% > path.backup
   ```
   3. 환경변수 PATH 수정 (주의: 반드시 1회만 실행할 것!!!)
   ```sh
   setx /m path "C:\VSCode\data\lib\python";"C:\VSCode\data\lib\python\Scripts";%PATH%
   ```
   4. 만약 환경변수 PATH를 잘못 수정했다면 백업을 이용해 복원 후 다시 시도
   ```sh
   type path.backup
   setx /m path {위 출력 내용 붙여넣기}
   ```

2. GUI 기반
   1. 실행창(Win+r)에서 sysdm.cpl 실행
   2. 고급 탭을 선택한 후 **환경 변수** 선택
   3. \<시스템 변수\> 목록에서 **Path** 항목을 찾아 더블클릭
   4. 목록 아래 빈 칸을 더블클릭한 후 다음 내용을 차례로 입력
   ```sh 
   C:\VSCode\data\lib\python
   ```
   ```sh 
   C:\VSCode\data\lib\python\Scripts
   ```   
   5. 추가한 2개 항목을 왼쪽 "위로 이동" 버튼을 이용해 목록 가장 앞으로 이동 
   ```sh
   C:\VSCode\data\lib\python
   C:\VSCode\data\lib\python\Scripts
   C:\WINDOWS\system32
   ...
   ```

3. [참고] 필수 경로
```sh
C:\WINDOWS\system32
C:\WINDOWS
C:\WINDOWS\System32\Wbem
C:\WINDOWS\System32\OpenSSH
C:\WINDOWS\System32\WindowsPowerShell\v1.0\
C:\Users\{계정이름}\AppData\Local\Microsoft\WindowsApps
```

4. CLI 환경을 재 시작하면 임의 경로에서 python 또는 pip 명령을 사용할 수 있음


### 코딩 폰트 다운로드 및 설치
> 코딩에 최적화된 폰트를 사용하면 소스 코드를 편집할 때 가독성이 높아짐.
>> 예를 들어 일반적인 폰트는 대문자 I와 소문자 l, 숫자 1을 구분하기 힘듬.

1. 다운로드 폴더에 임시 폴더를 만듦
2. 달서힐링체(DalseoHealingMedium.ttf)를 임시 폴더에 다운로드
   - [달서힐링체](https://github.com/webfontworld/daegu/raw/main/DalseoHealingMedium.ttf)
   - 0xProto와 어울리는 공개용 한글 폰트
3. 압축된 0xProto NF(0xProto.zip)를 다운받은 후 임시 폴더로 압축 해제
   - [0xProto NF](https://github.com/ryanoasis/nerd-fonts/releases/download/v3.2.1/0xProto.zip)
   - 영문/숫자 외에 NF는 수 많은 그래픽 문자 포함
4. 임시 폴더의 모든 .ttf를 선택한 후 우클릭, 메뉴가 표시되면 **설치** 선택


### 향상된 CLI 환경 구성
> 윈도우 10 이상에 기본 포함된 파워쉘(V1.x)보다 더 강력한 기능 제공
>> winget은 윈도우 패키지 관리자 서비스(msstore, winget)에 대한 클라이언트 툴

0. 윈도우 패키지 관리자인 winget 설치 및 업데이트 확인
   1. 윈도우 시작 메뉴에서 Microsoft Store 앱을 찾아 실행  
   2. 왼쪽 Activate Bar에서 라이브러리 선택 후 목록에서 Winget을 찾아 열기  
   3. 열린 winget 창에서 업데이트가 표시되면 업데이트 진행 및 완료될 때까지 대기  
   4. 만약 목록에 winget이 없으면 깃 허브의 microsoft/winget-cli에서 최신 버전의 설치 파일을 다운로드한 후 설치  
      - [microsoft/winget-cli](https://github.com/microsoft/winget-cli/releases)  
      - Assets 목록에서 확장자가 .msixbundle인 파일을 다운로드한 후 파일을 더블클릭해 설치  
   5. [주의] 윈도우 앱이 업데이트 중이거나 업데이트가 누락되면 진행 안됨  
    
1. cmd을 실행한 후 winget 버전 확인(v1.7.10861 이상)
```sh
winget --version
```
     
2. winget으로 해당 프로그램 설치
   - 설치 과정에서 관리자 권한을 요구하면 승인할 것  
   - 파워쉘과 같은 명령줄 쉘에 대한 최신 호스트 툴로 탭 분리, UTF-8, GPU 가속 텍스트 렌더링과 같은 다양한 기능 지원
   ```sh
   winget install Microsoft.WindowsTerminal -s winget
   ```
   - 파워쉘 7
   ```sh
   winget install Microsoft.PowerShell -s winget
   ```
   - 리눅스의 oh-my-zsh과 같은 파워쉘7용 테마 관리자
   ```sh
   winget install JanDeDobbeleer.OhMyPosh -s winget
   ```

3. 파워쉘7용 확장 모듈 설치
   1. PSGallery 저장소 인증
   ```sh
   Set-PSRepository -Name "PSGallery" -InstallationPolicy Trusted
   ```
   2. ls 명령 결과에 확장자에 따른 아이콘 추가   
   ```sh
   Install-Module -Name Terminal-Icons -Repository PSGallery
   ```
   3. 리눅스/맥의 oh-my-zsh과 같은 테마 적용
   ```sh
   install-module posh-git
   ```
   4. bat, lsd, dust, duf, broot, fd, sd, gping, xh, fzf와 같은 현대적인 유닉스/리눅스 호환 명령 추가
   ```sh
   Install-Module -Name modern-unix-win
   ```
   5. 파월쉘7에 fzf 연동
   ```sh
   Install-Module -Name PSFzf
   ```
   6. 설치된 모듈 확인
   ```sh
   Get-Module
   ```
   6. (옵션) 설치된 모듈 업데이트
   ```shㅣ
   Update-Module -Force
   ```

4. 새로 설치한 파워쉘7(pwsh)을 실행한 후 파워쉘7 모듈 경로 확인 
```sh
$env:PSModulePath -split ';'    
```
   - 홈 폴더(C:\Users\\<login-name\>) 아래 **Documents\PowerShell\Modules**
     - OneDrive에 문서폴더를 연결하지 않은 사용자로 홈 폴더 아래 OneDrive 폴더가 없음
   
   - 홈 폴더(C:\Users\\<login-name\>) 아래 **OneDrive\Documents\PowerShell\Modules** 
     -  OneDrive에 문서폴더를 연결 중이거나 했던 사용자로 홈 폴더 아래 OneDrive 폴더가 있음

5. 파워쉘7과 함께 실행된 윈도우 터미널의 설정창(Ctrl+,) 실행
   1. 시작 > 기본 프로필 > PowerShell
   2. 시작 > 기본 터미널 응용 프로그램 > Windows 터미널
   3. 프로필 > 기본값 > 모양 > 색 구성표 > Tango Dark
   4. 프로필 > 기본값 > 모양 > 글꼴 > 0xProto Nerd Font
   5. 프로필 > 기본값 > 모양 > 투명성 > 70%
   6. 프로필 > 기본값 > 고급 > 프로필 종료 동작 > 프로세스가 종료, 실패 또는 충돌 시 닫기
   7. 저장

6. 파워쉘7 시작 스크립트 작성
   1. 메모장으로 Microsoft.PowerShell_profile.ps1 파일을 열기
   ```sh
   notepad $profile
   ```
   2. 다음 내용을 열린 메모장에 복사해 붙여 넣은(앞서 설치한 모듈 로드 명령 추가) 후 저장 및 메모장 종료
   ```sh
   oh-my-posh init pwsh --config "$env:POSH_THEMES_PATH\quick-term.omp.json" | Invoke-Expression
   Import-Module -Name Terminal-Icons
   Import-Module -Name modern-unix-win
   Import-Module -Name PSFzf

   Set-PSReadlineOption –HistorySavePath ~\History.txt
   Set-PsFzfOption -PSReadlineChordProvider 'Ctrl+t' -PSReadlineChordReverseHistory 'Ctrl+r'

   function which ($command) {
      Get-Command -Name $command -ErrorAction SilentlyContinue | 
      Select-Object -ExpandProperty Path -ErrorAction SilentlyContinue
   }
   ```

<details>
<summary>Reference</summary>

*WinGet*
> Documentation (https://learn.microsoft.com/en-us/windows/package-manager/winget/)  
> GitHub (https://github.com/microsoft/winget-cli)

*Window Terminal*
> Documentation (https://learn.microsoft.com/en-us/windows/terminal/install)  
> GitHub (https://github.com/microsoft/terminal)  

*PowerShell*
> Documentation (https://learn.microsoft.com/en-us/powershell/)  
> GitHub (https://github.com/PowerShell/PowerShell)  

*OhMyPosh*
> Documentation (https://ohmyposh.dev/docs/installation/windows)  
> GitHub (https://github.com/JanDeDobbeleer/oh-my-posh)  
> Themes (https://ohmyposh.dev/docs/themes)
  
*Terminal Icons*
> GitHub (https://github.com/devblackops/Terminal-Icons)

*Nerd Fonts*
> WebSite (https://www.nerdfonts.com/font-downloads)

*PSReadline*
> Documentation (https://learn.microsoft.com/en-us/powershell/module/psreadline/)
> GitHub (https://github.com/PowerShell/PSReadLine)

*PSFzf*
> GitHub (https://github.com/kelleyma49/PSFzf)

*Modern Unix*
> GitHub (https://github.com/ibraheemdev/modern-unix)

  >> Core  
  cat -> bat (https://github.com/sharkdp/bat)  
  ls -> lsd (https://github.com/lsd-rs/lsd)  
  diff -> delta (https://github.com/dandavison/delta)  
  du -> dust (https://github.com/bootandy/dust)  
  du -> duf (https://github.com/muesli/duf)  
  cd -> zoxide (https://github.com/ajeetdsouza/zoxide)  

  >> Search  
  tree -> broot (https://github.com/Canop/broot)  
  sed -> sd (https://github.com/chmln/sd)  
  grep -> ripgrep(rg) (https://github.com/BurntSushi/ripgrep)  
  fzf (https://github.com/junegunn/fzf)

  >> System  
  ps -> procs (https://github.com/dalance/procs)  
  time -> hyperfine (https://github.com/sharkdp/hyperfine)  
  gtop/htop -> bottom(btm) (https://github.com/ClementTsang/bottom)  

  >> Network  
  ping -> gping (https://github.com/orf/gping)  
  dig -> dog (https://github.com/ogham/dog)  
  wget -> xh (https://github.com/ducaale/xh)  

  >> Etc  
  cheat (https://github.com/cheat/cheat)  
  
> Custom install  
  lazygit (https://github.com/jesseduffield/lazygit)
</details>

### VSCode 실행 및 Extensions 설치
> VSCode의 최초 상태는 단순 파일 편집 기능만 제공하며, Extensions을 통해 기능을 확장함.
>> MS는 VSCode Extensions 인터넷 저장소를 통해 전세계 개발자가 오픈소스로 개발한 Extensions 배포.
>> VSCode를 파이썬 통합개발환경으로 사용하려면 해당 Extensions 설치 필요.

1. 탐색기를 이용해 앞서 설치한 VSCode 루트에서 Code.exe 더블클릭
2. 왼쪽 Activity Bar에서 Extensions 선택
3. 상단 검색란에 해당 이름을 검색해 설치
   - Python: 파이썬 개발환경 지원
   - Jupyter: 주피터 노트북 지원
   - Python Indent: 파이썬 들여 쓰기 구분 지원
   - Remote - SSH: SSH 기반 원격 접속 지원
   - Tabnine: 인공지능 기반 코딩 지원(회원 가입이 필요하며 무료/유료로 구분. 일반적인 작업은 무료 버전으로도 가능)
   - Error Lens: 코드 오류의 원인을 주석으로 표시
   - Bookmarks: 특정 위치에 북마크를 추가한 후 필요할 때 한 번에 이동
   - Material Icon Theme: 아이콘 팩
   - Project Manager: 여러 작업 폴더 관리하며, 특정 작업 폴더를 빠르게 열어줌
   - Outline Map: 소스 파일에 포함된 함수, 클래스, 전역변수 목록 표시
   - One Dark Pro: 어두운 색상 테마
   - Ayu: 밝은 색상 테마
   - Korean Language Pack: 한국어 언어 팩  -> [주의] **모든 작업이 끝나기 전까지는 영문으로 설정할 것**
     - 실행 중 다른 언어 변경: F1 > configure display language 입력 후 선택 > 변경할 언어 선택
4. 단축키 \<Ctrl\>+, 를 눌러 Settings 표시 -> [참고] **설정은 변경 즉시 자동 저장됨**
   - 파일 자동 저장: Commonly Used > Files: Auto Save > onFocusChange
   - 폰트 설정: Commonly Used > Editor:Font family > '0xProto Nerd Font', DalseoHealing
     - 자동으로 한글은 달서달림, 그 외는 젯브레인모노 폰트가 적용됨
   - 마우스 휠로 편집창의 폰트 크기 확대/축소: 상단 검색란에 "wheel" 입력 > Editor: Mouse wheel Zoom 체크
5. 메뉴 > View > Appearance 클릭
   - Minimap 체크 해제
     - 편집 창에서 전체 소스 중 현재 위치를 표시하는 작은 창 제거
   - Secondary Side Bar 체크
   - 왼쪽 Activity Bar에서 Outline map Exension 아이콘을 끌어다 오른쪽 Secondary Side Bar에 놓기

### 결과 확인
1. 문자열 기반 파이썬 명령 실행
   ```sh
   python -c "import this"
   python -c "import antigravity"
   ```
[옵션]
2. 설치된 모든 파이썬 패키지 기록
   ```
   pip freeze > requirements.txt
   ```
3. 설치 기록을 토대로 동일한 패키지 재 설치
   ```
   pip install -r requirements.txt
   ```   
4. 설치된 모든 파이썬 패키지를 최신 버전으로 업그레이드
    ```sh
    pip freeze | %{$_.split('==')[0]} | %{pip install --upgrade $_}
    ```
