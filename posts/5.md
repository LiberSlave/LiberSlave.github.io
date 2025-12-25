
## github
현재 까지 작업사항을 깃허브에 올린다.
백업차원 및, 구글Jules와 [코드베이스분석 AI](https://github.com/The-Pocket/PocketFlow-Tutorial-Codebase-Knowledge) 같은 오픈소스를 이용해 보고 싶어서다.

```
git remote add origin https://github.com/LiberSlave/office-add-in-trading.git
git push -u origin main
git push -u origin main
```

1. **GitHub 저장소를 원격(remote) 저장소로 추가합니다.**
	 컴퓨터(로컬)에 있는 Git 프로젝트에게 "저기 GitHub라는 인터넷 서버에 네 코드의 복사본을 저장하고 공유할 수 있는 공간이 있어. 그 공간의 주소는 이거고, 앞으로 그 공간을 'origin'이라고 부를게." 라고 알려주는 과정입니다.
2. 기본 브랜치 이름을 main으로 변경합니다.
	최근에 기본 브랜치가 master -> main으로 변경되고 있다고 한다.
3. **로컬 저장소의 내용을 GitHub 원격 저장소로 푸시(push)합니다.**


```
git push
```
- **대부분의 경우, 특히 이전에 git push -u origin <브랜치이름>으로 한 번이라도 푸시한 적이 있다면, git push만 입력해도 현재 작업 중인 로컬 브랜치의 변경 사항이 연결된 원격 브랜치로 잘 푸시됩니다.**


## codebase 분석 오픈소스 

[코드베이스분석 AI](https://github.com/The-Pocket/PocketFlow-Tutorial-Codebase-Knowledge)

1. 원하는 위치에 프로젝트 폴더를 생성한다.
```
git clone https://github.com/The-Pocket/PocketFlow-Tutorial-Codebase-Knowledge cd PocketFlow-Tutorial-Codebase-Knowledge
```
2. uv 가상환경을 생성하고 활성화 한다.
```
# 가상 환경 생성 (예: .venv 폴더에) 자동으로 .venv폴더 생성됨
uv venv
# 가상 환경 활성화
.venv\Scripts\Activate.ps1
```

3. 의존성 설치 (requirements.txt 파일에 명시된 의존성 패키지들을 설치함.)
```
uv pip install -r requirements.txt
```

4. 코드 실행(llm설정 확인)
```
python utils/call_llm.py
uv run utils/call_llm.py
```
만약 가상환경이 실행되어 있다면 두 명령어는 같은 행동임. 

5. 로컬 디렉토리의 코드베이스 분석해보기 (파이썬)
```
python main.py --dir "C:\knowledge\1. Project\주식\복기\My Office Add-in" --include "*.py" --exclude "*test*"
```

6. 다른 언어도 함께 분석(파이썬 + 자바스크립트)
```
python main.py --dir "C:\knowledge\1. Project\주식\복기\My Office Add-in" --include "*.py" "*.js" --exclude "*test*"
```
