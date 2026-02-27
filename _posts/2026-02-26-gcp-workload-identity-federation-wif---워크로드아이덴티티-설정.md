---
title: GCP Workload Identity Federation (WIF - 워크로드아이덴티티) 설정
author: Seong Gi
date: 2026-02-26
categories: [Cloud, GCP]
tags: [gcp, wif, keycloak, oidc, security]
---

## GCP Workload Identity Federation (WIF - 워크로드아이덴티티) 설정  
<br>

```
개요 : 
서비스 계정 키 대신 GCP 워크로드 아이덴티티 사용방안

외부 신원 공급자(IdP)와 구글 클라우드 간의 신뢰 관계를 구축하여, "Keyless 인증" 환경을 실현하는 것을 목적으로 합니다.
기존 방식의 한계 극복: 외부 연동 시 보안 취약점이 될 수 있는 정적 자격 증명(SA Key)의 생성 필요성을 원천적으로 제거합니다.
운영 오버헤드 감소: 주기적인 키 로테이션, 만료 관리, 폐기 절차 등의 수동 관리 업무를 자동화된 토큰 교환 방식으로 대체합니다.
통합 신원 관리 (IdP Integration): Keycloak, Okta, AD 등 기존에 사용 중인 중앙 집중식 인증 시스템을 활용하여 클라우드 자원에 대한 액세스를 통제합니다.
규정 준수 (Compliance): 장기 자격 증명 사용을 금지하는 국내외 보안 컴플라이언스 및 거버넌스 요구 사항을 충족합니다.
```

<br>

<br>

![](/assets/images/posts/gcp-workload-identity-federation-wif---워크로드아이덴티티-설정/image.png)<br>

- Service Account 에서 Key 부분에서 구글이 SA키 다운로드를 하지말고, 워크로드 아이덴티티 제휴 사용을 권고하는 내용

![](/assets/images/posts/gcp-workload-identity-federation-wif---워크로드아이덴티티-설정/image%202.png)<br>

<br>

- GCP 워크로드 아이덴티티 제휴 설정방안  
IAM 및 관리자 / 워크로드 아이덴티티 제휴 이동  
![](/assets/images/posts/gcp-workload-identity-federation-wif---워크로드아이덴티티-설정/image%203.png)  
  
  

<br>

![](/assets/images/posts/gcp-workload-identity-federation-wif---워크로드아이덴티티-설정/image%204.png)<br>

<br>

### Step 1: Workload Identity Pool 생성

1. **메뉴 경로**: `IAM 및 관리` > `Workload Identity Federation`
2. **\[풀 만들기\]** 클릭.
3. **이름**: `wif-keycloak-pool`
4. **설명**: `Keycloak 연동을 위한 외부 인증 풀`

<br>
![](/assets/images/posts/gcp-workload-identity-federation-wif---워크로드아이덴티티-설정/image%205.png)<br>
<br>

### Step 2: OIDC 공급자(Provider) 추가

예시로는 오픈소스인 keycloak에서 발급한 정보를 토대로 작성하지만, okta등 OIDC 공급자에서 제공하는 정보를 입력하면 됩니다.

1. 생성한 풀 내부에서 **\[공급자 추가\]** 클릭.
2. **공급자 선택**: `OpenID Connect (OIDC)`
3. **공급자 이름**: `keycloak-provider`
4. **Issuer(발급기관) URL**: **ngrok 주소** 입력 (예: `https://<random>.ngrok-free.app/realms/GCP-WIF-Realm`)|아니면 okta URL등 공급자 URL을 입력합니다.
5. **대상(Audience)**: Keycloak에서 만든 **Client ID** 입력 (`gcp-wif-client`)  
  

![](/assets/images/posts/gcp-workload-identity-federation-wif---워크로드아이덴티티-설정/image%206.png)<br>

### Step 3: 속성 매핑(Attribute Mapping)

1. 구글이 Keycloak의 정보를 어떻게 이해할지 정의합니다.
    - **google.subject** = `assertion.sub`
    - (선택사항) **attribute.email** = `assertion.email`

## <br>

## IAM 서비스 계정 및 권한 대행(Impersonation) 설정

### Step 1: 전용 서비스 계정 생성

1. **메뉴 경로**: `IAM 및 관리` > `서비스 계정`
2. **\[서비스 계정 만들기\]** 클릭.
3. **이름**: `gemini-api-test-sa`
4. **역할 부여**: `Vertex AI 사용자` (`roles/aiplatform.user`)

만약 워크로드 풀에 직접 액세스 권한을 준다면, 아래와 같은 풀로 직접 IAM에서 권한을 부여합니다.  
`principalSet://iam.googleapis.com/projects/<프로젝트번호>/locations/global/workloadIdentityPools/<풀이름>/*`  

![](/assets/images/posts/gcp-workload-identity-federation-wif---워크로드아이덴티티-설정/image%207.png)<br>

<br>

### Step 2: WIF와 서비스 계정 연결

1. keycloak 계정에 직접 권한 부여시
2. 생성한 서비스 계정 상세 페이지 진입 > 액세스 권한이 있는 주 구성원 탭 클릭.
3. **\[액세스 권한 부여\]** 클릭.
4. **새 주 구성원**: WIF 풀의 식별자를 입력합니다.
    - 형식: `principalSet://iam.googleapis.com/projects/<프로젝트번호>/locations/global/workloadIdentityPools/<풀이름>/*   `

5. **역할 선택**: Vertex AI 사용자

<br>

**서비스 가장(서비스계정)선택시** 

서비스계정 → 액세스 권한이 있는 주 구성원 → 계정에 추가

형식은 동일

`principalSet://iam.googleapis.com/projects/<프로젝트번호>/locations/global/workloadIdentityPools/<풀이름>/*`  

**![](/assets/images/posts/gcp-workload-identity-federation-wif---워크로드아이덴티티-설정/image%208.png)<br>
**

**서비스 계정 토큰 생성자** (`roles/iam.serviceAccountTokenCreator`) 권한 부여

<br>

|     |     |     |
| --- | --- | --- |
| **구분** | **1\. 제휴 ID 방식 (권장)** | **2\. 서비스 계정 가장 방식** |
| **핵심 개념** | 외부 SSO ID(`test-user`)가 직접 권한을 가짐 | 외부 ID가 서비스 계정의 권한을 빌려 씀 |
| **GCP 인식** | `principalSet://.../*`가 주체임 | `service-account@...`가 주체임 |
| **IAM 설정 위치** | **프로젝트 IAM** 메뉴 | **서비스 계정 상세** 메뉴 |
| **필요 역할** | 리소스 사용 권한 (예: Vertex AI 사용자) | **서비스 계정 토큰 생성자** (`roles/iam.serviceAccountTokenCreator`) |
| **JSON 특징** | `service_account_impersonation_url` 없음 | `service_account_impersonation_url` 포함 |

<br>

<br>

<br>

<br>

![](/assets/images/posts/gcp-workload-identity-federation-wif---워크로드아이덴티티-설정/image%209.png)<br>

- 서비스계정 가장 선택시 서비스 계정이 선택되어 있음

![](/assets/images/posts/gcp-workload-identity-federation-wif---워크로드아이덴티티-설정/image%2010.png)<br>

<br>

- 워크로드 풀에 직접 액세스 권한을 부여시서비스 계정 표현 안됨

<br>

![](/assets/images/posts/gcp-workload-identity-federation-wif---워크로드아이덴티티-설정/image%2011.png)<br>

<br>

제공자 : 기존 설정한 “keycloak-provider” 이 존재하며 여러 제공자를 만들었다면 해당 폴에 맞는 제공자를 선택

OIDC ID token 경로 : 아무곳이나 해도 상관없으며, 본 문서에서는 tmp에 임시저장으로 설정

형식 유형 : text, json 모두 가능 (별도 설정 필요없음)

<br>

PSC 테스트 코드

동작시 아래와 같이 진행되며

11개의 입력 값을 넣어야 합니다.

<br>

![](/assets/images/posts/gcp-workload-identity-federation-wif---워크로드아이덴티티-설정/image%2012.png)<br>

<br>

코드 선택 방안은 아래와 같습니다.

<br>

\[인증 방식 선택\]  
 1. 구성 파일 방식: JSON 파일 내 정의된 가장(Impersonation) 설정 사용  
 2. 서비스 가장 방식: 코드에서 직접 대상 서비스 계정을 명시

<br>
 • AUTH\_METHOD \[1\]: 2  
 • SERVICE\_ACCOUNT [\[gemini-api-key-test@ctu-gcp-dsa2-unit.iam.gserviceaccount.com\]:](mailto:[gemini-api-key-test@ctu-gcp-dsa2-unit.iam.gserviceaccount.com]:)   
 • PSC\_ENDPOINT\_IP \[10.20.30.33\]:   
 • PROJECT\_ID \[ctu-gcp-dsa2-unit\]:   
 • LOCATION \[us-central1\]:   
 • MODEL\_ID \[gemini-2.0-flash\]:   
 • KC\_URL \[[https://yoko-lakier-kenny.ngrok-free.dev\]:](https://yoko-lakier-kenny.ngrok-free.dev]:)   
 • KC\_REALM \[GCP-WIF-Realm\]:   
 • KC\_CLIENT\_ID \[gcp-wif-client\]:   
 • KC\_USER \[test-user\]:   
 • KC\_PW \[\*\*\*\*\*\*\*\*\] (변경 시 입력): 

<br>

소스코드

```
import sys
import subprocess
import os
import json
import warnings
import getpass

# --- [1. 가상 환경(Venv) 자동 구성 및 전환 로직] ---
def ensure_venv():
    """스크립트 실행에 필요한 가상 환경을 자동으로 생성하고 라이브러리를 설치합니다."""
    VENV_DIR = os.path.join(os.getcwd(), ".venv")
    
    # 현재 실행 중인 파이썬이 가상 환경인지 확인
    if sys.prefix == sys.base_prefix:
        if not os.path.exists(VENV_DIR):
            print(f"📦 가상 환경(.venv) 구성을 시작합니다...")
            try:
                subprocess.check_call([sys.executable, "-m", "venv", VENV_DIR])
            except:
                print("❌ venv 생성 실패. 'sudo apt install python3-venv'가 필요할 수 있습니다.")
                sys.exit(1)
        
        # 가상 환경 내 파이썬 실행 파일 경로 설정
        python_bin = os.path.join(VENV_DIR, "bin", "python") if os.name != 'nt' else os.path.join(VENV_DIR, "Scripts", "python.exe")
        print("📥 가상 환경 내 필수 라이브러리(requests, google-auth) 설치 중...")
        subprocess.check_call([python_bin, "-m", "pip", "install", "--upgrade", "pip", "requests", "google-auth"])
        
        print("🚀 가상 환경으로 전환하여 스크립트를 재실행합니다.\n")
        env = os.environ.copy()
        env["VIRTUAL_ENV"] = VENV_DIR
        # 가상 환경의 파이썬으로 현재 파일 다시 실행
        subprocess.call([python_bin] + sys.argv, env=env)
        sys.exit()

# --- [인코딩 문제 해결을 위한 안전한 입력 함수] ---
def safe_input(prompt_text):
    """터미널 인코딩에 상관없이 한국어를 안전하게 입력받습니다."""
    sys.stdout.write(prompt_text)
    sys.stdout.flush()
    line = sys.stdin.buffer.readline()
    for enc in ['utf-8', 'cp949', 'euc-kr']:
        try:
            return line.decode(enc).strip()
        except UnicodeDecodeError:
            continue
    return line.decode('utf-8', errors='replace').strip()

# 스크립트 시작 시 최우선으로 가상 환경 체크
if __name__ == "__main__":
    ensure_venv()

# --- [2. 필수 라이브러리 임포트] ---
import requests
import google.auth
import google.auth.impersonated_credentials
import google.auth.transport.requests
from urllib3.exceptions import InsecureRequestWarning

# --- [3. 설정 및 환경 구성] ---
CONFIG_FILE = "gemini_config.json"
TOKEN_PATH = "/tmp/keycloak_token.txt"

DEFAULT_CONFIG = {
    "AUTH_METHOD": "2", # 1: 제휴 ID(ADC), 2: 서비스 계정 가장
    "WIF_CONFIG_FILE": "client_library_config.json",
    "SERVICE_ACCOUNT": "gemini-api-key-test@ctu-gcp-dsa2-unit.iam.gserviceaccount.com",
    "PSC_ENDPOINT_IP": "10.20.30.33",
    "PROJECT_ID": "ctu-gcp-dsa2-unit",
    "LOCATION": "us-central1",
    "MODEL_ID": "gemini-2.0-flash",
    "KC_URL": "https://yoko-lakier-kenny.ngrok-free.dev",
    "KC_REALM": "GCP-WIF-Realm",
    "KC_CLIENT_ID": "gcp-wif-client",
    "KC_USER": "test-user",
    "KC_PW": "Ahdehf5@30"
}

def load_or_ask_config():
    config = DEFAULT_CONFIG.copy()
    if os.path.exists(CONFIG_FILE):
        try:
            with open(CONFIG_FILE, "r", encoding="utf-8") as f:
                config.update(json.load(f))
            print(f"\n📂 기존 설정 파일({CONFIG_FILE})을 불러왔습니다.")
        except: pass

    print("\n=========================================")
    print("🔧 [현재 설정값 확인]")
    for k, v in config.items():
        # 방식 1일 때 서비스 계정은 출력에서도 숨겨서 깔끔하게 유지
        if config["AUTH_METHOD"] == "1" and k == "SERVICE_ACCOUNT": continue
        display_v = "********" if k == "KC_PW" else v
        print(f" • {k:18}: {display_v}")
    print("=========================================")

    choice = safe_input("\n👉 이 설정대로 진행하시겠습니까? (Y/n) [기본값: Y]: ").lower()
    if choice not in ["", "y", "yes"]:
        print("\n📝 설정을 변경합니다. (엔터=기존값 유지)")
        
        print("\n[인증 방식 선택]")
        print(" 1. 제휴 ID 방식: JSON 파일 기반 (IAM 권한 직접 부여 필요)")
        print(" 2. 서비스 가장 방식: 코드 명시 방식 (Impersonation 권한 필요)")
        new_method = safe_input(f" • AUTH_METHOD [{config['AUTH_METHOD']}]: ")
        if new_method: config["AUTH_METHOD"] = new_method

        for key in config.keys():
            if key == "AUTH_METHOD": continue
            if config["AUTH_METHOD"] == "1" and key == "SERVICE_ACCOUNT": continue
            
            if key == "KC_PW":
                # 비밀번호 입력 시 화면 노출 차단
                user_input = getpass.getpass(f" • {key} [********] (변경 시 입력): ").strip()
            else:
                user_input = safe_input(f" • {key} [{config[key]}]: ")
            
            if user_input: config[key] = user_input
        
        with open(CONFIG_FILE, "w", encoding="utf-8") as f:
            json.dump(config, f, indent=4)
        print(f"✅ 설정이 '{CONFIG_FILE}'에 저장되었습니다.")
    
    return config

# --- [4. Keycloak 및 GCP WIF 인증 로직] ---
def get_keycloak_token(config):
    url = f"{config['KC_URL']}/realms/{config['KC_REALM']}/protocol/openid-connect/token"
    payload = {
        "grant_type": "password",
        "client_id": config['KC_CLIENT_ID'],
        "username": config['KC_USER'],
        "password": config['KC_PW'],
        "scope": "openid"
    }
    try:
        print(f"🔑 Keycloak 토큰 요청 중...")
        resp = requests.post(url, data=payload, timeout=10)
        if resp.status_code != 200:
            print(f"❌ Keycloak 오류 ({resp.status_code}): {resp.text}")
            return False
        token = resp.json().get("id_token")
        with open(TOKEN_PATH, "w") as f: f.write(token)
        return True
    except Exception as e:
        print(f"❌ Keycloak 연결 실패: {e}")
        return False

def get_access_token(config):
    if not get_keycloak_token(config): return None
    
    wif_file = config["WIF_CONFIG_FILE"]
    if not os.path.exists(wif_file):
        print(f"❌ WIF 구성 파일을 찾을 수 없습니다: {wif_file}")
        return None
        
    os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = wif_file
    scopes = ["https://www.googleapis.com/auth/cloud-platform"]
    
    try:
        source_creds, _ = google.auth.default(scopes=scopes)
        if config["AUTH_METHOD"] == "2":
            print(f"☁️ [방식 2] 서비스 계정({config['SERVICE_ACCOUNT']}) 가장 수행 중...")
            creds = google.auth.impersonated_credentials.Credentials(
                source_credentials=source_creds,
                target_principal=config['SERVICE_ACCOUNT'],
                target_scopes=scopes,
                lifetime=3600
            )
        else:
            print(f"☁️ [방식 1] 구성 파일({wif_file}) 기반 인증 중...")
            creds = source_creds
            
        creds.refresh(google.auth.transport.requests.Request())
        return creds.token
    except Exception as e:
        print(f"❌ GCP 토큰 교환 실패: {e}")
        return None

# --- [5. Gemini API 호출 로직] ---
def ask_gemini(config, prompt):
    token = get_access_token(config)
    if not token: return

    api_url = f"https://{config['PSC_ENDPOINT_IP']}/v1/projects/{config['PROJECT_ID']}/locations/{config['LOCATION']}/publishers/google/models/{config['MODEL_ID']}:generateContent"
    headers = {
        "Authorization": f"Bearer {token}",
        "Content-Type": "application/json",
        "Host": f"{config['LOCATION']}-aiplatform.googleapis.com"
    }
    body = {"contents": [{"role": "user", "parts": [{"text": prompt}]}]}

    print(f"\n🚀 Gemini 요청 전송 (PSC: {config['PSC_ENDPOINT_IP']})...")
    warnings.filterwarnings('ignore', category=InsecureRequestWarning)
    try:
        response = requests.post(api_url, headers=headers, json=body, verify=False, timeout=30)
        response.raise_for_status()
        print("\n🤖 --- Gemini 응답 ---")
        print(response.json()["candidates"][0]["content"]["parts"][0]["text"])
        print("-----------------------")
    except Exception as e:
        print(f"\n❌ API 호출 오류: {e}")
        if 'response' in locals() and response.status_code == 403:
            print("💡 팁: 인증 방식에 따른 IAM 권한 설정을 다시 확인하세요.")
    finally:
        warnings.resetwarnings()

# --- [메인 실행부] ---
if __name__ == "__main__":
    final_config = load_or_ask_config()
    user_prompt = safe_input("\n💬 질문 입력 [엔터=자기소개]: ") or "자기소개 부탁해"
    ask_gemini(final_config, user_prompt)
```