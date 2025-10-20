## tasks.json

LS증권의 api를 이용하기 위해서는 access_tocken이 필요하다. 
이 토큰은 하루한번 새로 발급받아야한다.
또한 주식 종목이름과 종목코드를 매칭시킨 ticker.json이 내 로컬에 저장되어있는데 이또한 api로 종목을 가져와서 하루에 한번 업데이트 해야한다. 매일같이는 아니지만 종종 신규주가 추가되기 때문이다.
따라서 이에 대한 코드는 새로 스크립트에 저장하여 작업 스케줄러를 통해 매일 하루에 한번 실행하고자 한다.

이 tocken발급과 ticker.json을 업데이트 하는 함수는 이미 구현이 되어있고 새로운 py파일에 이에 대한 코딩을 했다. 
그리고 .vscode\tasks.json에 다음과 같이 추가하여 프로젝트 개발서버를 시작할때(나같은 경우 아침에 한번 시작한다.) 실행되도록 했다.
``` json
{

      "label": "Run Daily Update",

      "type": "shell",

      "command": "uv",

      "args": [

        "run",

        "update_once_a_day.py"

      ],

      "options": {

        "cwd": "${workspaceFolder}/src/fapi"

      },

      "presentation": {

        "clear": true,

        "panel": "dedicated"

      },

      "problemMatcher": []

    },

    {

      "label": "Start Full Development Environment",

      "dependsOrder": "parallel",

      "dependsOn": [

        "Start Backend Server",

        "Debug: Excel Desktop",

        "Run Daily Update"

      ],

      "presentation": {

        "clear": true

      }

    }
```


ctrl+shift+p 후 다음 절차를 따르면 스크립트들이 실행된다.

![[Pasted image 20250621165222.png]]![[Pasted image 20250621165233.png]]