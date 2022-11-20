# 클린 코드란?
≠ 짧은 코드

== `원하는 로직을 빠르게 찾을 수 있는 코드`

# 클린코드를 위한 세 가지 요소

- `응집도`
- `단일책임`
- `추상화`

> ## 응집도

- 하나의 목적을 가진 코드들은 위치적으로 붙어있어야 한다. (흩뿌려져 있으면 유지보수 시 어려움 발생)
- 팝업을 조작하는 코드들이 **뚝뚝 떨어져 있어서** `파악도 한 번에 안 되고 버그 발생 위험도 높다`.
    
    ```jsx
    // 노란색 표시 -> 팝업을 조작하는 코드
    function QuestionPage() {
    	const [popupOpend, setPopupOpened] = useState(false);
    	async function handleClick() {
    		setPopupOpened(true);
    	};
    	function handlePopupSubmit() {
    		await 질문전송(연결전문가.id);
    		alert("질문을 전송했습니다");
    	}
    
    	return (
    		<>
    			<button onClick={handleClick}>질문하기</button>
    			<Popup title="보험 질문하기" open={popupOpened}>
    				<div>전문가가 설명해드려요></div>
    				<button onClick={handlePopupSubmit}>확인</button>
    			</Popup>
    		</>
    	);
    }
    ```
    

### Tip1) Custom hook 을 사용하여 응집도 높이기

- Custom hook(`useMyExpertPopup()`) 을 사용해서 한 군데로 뭉침.
    
    ```jsx
    function QuestingPage() {
    	const [openPopup] = useMyExpertPopup(연결전문가.id);
    
    	function handleClick(){
    		openPopup();
    	};
    
    	return <button onClick={handleClick}>질문하기</button>
    }
    ```
    
- 하지만 오히려 Custom hook 함수에 가려서 아래의 내용에 대한 코드 파악이 어려워짐.
    - 어떤 내용의 팝업을 띄우는지?
    - 팝업에서 버튼 클릭 시, 어떤 액션을 하는지?
- 따라서, 뭉쳐야 될 코드를 한 마디로 정리하자면 ‘`당장 몰라도 되는 디테일`’  이다. 그러면 짧은 코드를 보더라도 빠르게 코드의 목적을 파악하는 것이 쉬워진다. 
반대로 코드 파악에 ‘`필수적인 핵심 정보`’ 에 대한 코드를 뭉쳐서 숨겨놓으면 여러 모듈을 넘나들며 흐름을 따라가야 하는 대참사가 발생하니 주의하도록 한다.

- **결론)**
    1. 먼저 남겨야 할 `핵심 데이터`와 `숨겨도 될 세부 구현`을 나누기
        
        ```jsx
        // 노란색 표시 -> 팝업을 조작하는 코드
        function QuestionPage() {
        	const [popupOpend, setPopupOpened] = useState(false);
        	async function handleClick() {
        		setPopupOpened(true);
        	};
        	function handlePopupSubmit() {
        		await 질문전송(연결전문가.id);
        		alert("질문을 전송했습니다");
        	}
        
        	return (
        		<>
        			<button onClick={handleClick}>질문하기</button>
        			<Popup title="보험 질문하기" open={popupOpened}>
        				<div>전문가가 설명해드려요></div>
        				<button onClick={handlePopupSubmit}>확인</button>
        			</Popup>
        		</>
        	);
        }
        ```
        
        - 위의 코드에서 **핵심 데이터**는 `팝업 클릭 시 수행하는 액션`, `팝업 제목`, 그리고 `내용` 
        **세부 구현**은 `팝업을 열고 닫을 때 사용하는 상태(state), 컴포넌트의 세부 마크업, 팝업 버튼 클릭 시, 특정 함수를 호출해야 한다는 바인딩` 이다.
    2. 나눈 `핵심 데이터`, `세부구현` 을 토대로 코드를 다시 리팩터링 하기
        
        ```jsx
        // 노란색 표시 -> 팝업을 조작하는 코드
        function QuestionPage() {
        	const [openPopup] = usePopup(연결전문가.id);
        	~~const [popupOpend, setPopupOpened] = useState(false);~~
        	async function handleClick() {
        		const confirmed = await openPopup({
        			title: "보험 질문하기",  // 팝업 제목
        			contents: <div>전문가가 설명해드려요</div>, // 팝업 내용
        		});
        		if (confirmed) {
        			await submitQuestion();  // 팝업 버튼 Action
        		}
        	};
        ~~~~
        	function submitQuestion(연결전문가) {
        		await 질문전송(연결전문가.id);
        		alert("질문을 전송했습니다");
        	}
        
        	return (
        		<>
        			<button onClick={handleClick}>질문하기</button>
        			~~<Popup title="보험 질문하기" open={popupOpened}>
        				<div>전문가가 설명해드려요></div>
        				<button onClick={handlePopupSubmit}>확인</button>
        			</Popup>~~
        		</>
        	);
        }
        ```
        
        - 즉 openPopup 이라는 Custom hook 안에 모든 코드를 다 숨기는 게 아니라 세부 구현만 숨겨놓고 핵심 데이터인 `팝업 제목, 내용, 액션`은 바깥에서 넘기는 것이다.
        이렇게 하면 세부구현을 읽지 않고도 어떤 팝업인지 파악하기가 쉽다.
        - 이 방법이 ‘`선언적 프로그래밍`’ 이라고 한다.

> ## 단일 책임

- 하나의 함수는 하나의 작업만 하도록 한다. (하나의 함수에서 여러가지 작업을 할 경우, 유지보수 시 어려움 발생)

```jsx
async function handle질문제출() {
	const 약관동의 = await 약관동의_받아오기();
	if (!약관동의) {
		await 약관동의_팝업열기();
	}
	await 질문전송(questionValue);
	alert ("질문이 등록되었어요.");
}
```

- 위와 같이 **폼에서 질문 제출 버튼을 클릭했을 때 호출되는 함수명**을 `handle질문제출()` 으로 하면 어떨까?
- 결론적으로는 좋지 않은 함수명이다. 그 이유는 **함수명**은 ‘`질문제출`’ 인데, **내용**에는 ‘`약관제출`’, ‘`질문제출`’ 이 섞여있기 때문이다.
    
    ```jsx
    // handle질문제출() 이라는 함수 안에, 질문 제출 기능 외 '약관 체크 후 팝업 관리' 기능도 포함되어 있어서 bad code!
    async function handle질문제출() {
    	const 약관동의 = await 약관동의_받아오기();    // 약관 체크 후 팝업
    	if (!약관동의) {
    		await 약관동의_팝업열기();
    	}
    	await 질문전송(questionValue);    // 질문 제출
    	alert ("질문이 등록되었어요.");
    }
    ```
    
- 따라서, 중요 포인트가 모두 담겨 있지 않은 함수명은 `위험` 할 수 있다. 이렇게 중요 포인트가 모두 담겨있지 않은 함수명은 읽는 이가 예상한 대로 코드가 작동하지 않으며, 이는 코드에 대한 신뢰 하락으로 이어진다. 그 다음부터는 함수명을 믿지 못하고 세부 구현을 모두 의심 가득한 눈초리로 보게 될 것이다.
- 여기에 신규 기능까지 추가 시, 코드는 더욱 **작성자만** **유지보수를 할 수 밖에 없는 구조로 구현**될 것이니 주의하자!!
    
    ```jsx
    // 신규 기능이 추가할 경우 (+연결전문가 질문 제출 기능)
    async function handle질문제출() {
    	const 약관동의 = await 약관동의_받아오기();    // 약관 체크 후 팝업
    	if (!약관동의) {
    		await 약관동의_팝업열기();
    	}
    	await 질문전송(questionValue);    // 질문 제출
    	alert ("질문이 등록되었어요.");
    
    +	const 연결전문가 = await 연결전문가_질문전송(questionValue);  // + 연결전문가 질문 제출
    +	if (연결전문가 !== null) {
    +		await 연결전문가_질문전송(questionValue);
    +		alert(`${연결전문가.name}에게 질문이 등록되었어요.`);
    	}
    }
    ```
    

### Tip1) 한 가지 일만 하는 함수 - **함수명에 대한 신뢰도 높이기**

- 위에 `handle질문제출()` 함수에서 이뤄지는 세 가지 작업들을 각각 별도 함수들로 세분화시켰다.
이렇게 하면, 각 함수명에 대한 신뢰도 ⇪

```jsx
// good code
// 한가지 작업만 하는
async function handle연결전문가질문제출() {
	await 연결전문가_질문전송(questionValue);
	alert(`${연결전문가.name}에게 질문이 등록되었어요.`);
}

async function handle새전문가질문제출() {
	await 질문전송(questionValue);    // 질문 제출
	alert ("질문이 등록되었어요.");
}

async function handle약관동의팝업() {
	const 약관동의 = await 약관동의_받아오기();    // 약관 체크 후 팝업
	if (!약관동의) {
		await 약관동의_팝업열기();
	}
```

### Tip2) 한 가지 일만 하는 기능성 컴포넌트 - **리액트 컴포넌트로 기능을 분리하기**

- 아래와 같이 버튼을 클릭하면 서버에 로그를 찍는 코드가 있다. 여기서 아쉬운 점은 버튼 클릭 함수에 로그 찍는 함수와  API call 이 섞여있다는 것이다.
    
    ```jsx
    // before
    
    const MainComponent = () => {
    	return (
    		<button
    			onClick={async () => {
    				log('제출 버튼 클릭')
    				await openConfirm();
    			}}
    		/>
    	)
    };
    ```
    
- 이는 `LogClick 이라는 컴포넌트를 만들어서 버튼을 감싸게 하고` 버튼을 클릭하면 자동으로 클릭 로그가 전송되도록 리팩터링 하는 방법이 있다. 이렇게 리팩터링 하면 버튼 클릭함수에서는  API call 만 신경 쓸 수 있는 장점이 있다.
    
    ```jsx
    // after
    const LogClick = ({message}) => {
    	log(message)
    	return;
    }; 
    
    const MainComponent = () => {
    	return (
    		<LogClick message="제출 버튼 클릭">
    			<button onClick={openConfirm()}/>
    		</LogClick>
    	)
    }
    ```
    

### Tip3) 상황에 따라 한글 네이밍도 사용해보기**- 영어 네이밍이 길어질 경우, 유용할 수 있음**

- 소소한 팁 중 하나로, 아래와 같은 상황에서는 오히려 한글 네이밍을 사용해보면 쉽게 대처할 수 있고 주석을 달아둔 것과 같은 효과까지 얻을 수 있다.
    - ’실시간 상담 완료’ 나 ‘연결 고객 5명 미만’ 등 도메인이 복잡해서 `영어 이름을 길게 짓는게 오히려 복잡도를 높일 때`
    - 상수를 `직관적으로 보고 싶을 때`
    - `복잡한 조건문`이 많아질 때
    
    ```jsx
    const 패널티풀림 = reasons.indexOf('PENALTY') > -1;
    const 평점4점이상 = review.rate >= 80;
    
    if (패널티풀림) {
    	return //...
    }
    if (평점4점이상) {
    	return //...
    }
    ```
    
    ```jsx
    const 설계사정보팝업_노출 = 12345;
    const 설계사정보팝업_확인 = 54321;
    
    const handleMatchPlanner = async () => {
    	log(설계사정보팝업_노출);
    	const confirmed = await openConfirm();
    	if (confirmed) {
    		log(설계사정보팝업_확인);
    		goToChat(PlannerId);
    	}
    };
    ```
    

> ## 추상화

- 함수의 세부구현 단계가 제각각일 때, 추상화 단계를 조정해서 `핵심 개념`을 필요한 만큼만 노출해야 한다.

### Tip1) 프런트엔드 코드의 추상화 - 컴포넌트

- 팝업 기능 코드
    - 팝업 코드 제로부터 구현
        
        ```jsx
        // before
        <div style={팝업스타일}>
        	<button onClick={async () => {
        		const res = await 회원가입();
        		if (res.success) {
        			프로필로이동();
        		}
        	}}>
        		전송
        	</button>
        </div>
        ```
        
    - 중요 개념만 남기고 `추상화`
        
        ```jsx
        // after
        
        <Popup
        	onSubmit={회원가입}
        	onSuccess={프로필로이동}
        />
        ```
        

### Tip2) 프런트엔드 코드의 추상화 - 함수

- 설계사 라벨을 얻는 코드
    - 코드 세부 구현
        
        ```jsx
        // before
        const planner = await fetchPlanner(plannerId);
        const label = planner.new ? '새로운 상담사' : '연결 중인 상담사';
        ```
        
    - 중요 개념을 `함수 이름에 담아 추상화`
        
        ```jsx
        // after
        const label = await getPlannerLabel(plannerId);
        ```
        

### 상황에 따른 추상화 Level

- 추상화  level 에 정답은 없다. 단지 `상황에 따라 필요한 만큼 추상화`하면 된다.
- 예시) 전송 버튼 클릭 시, 성공 메시지를 띄우는 코드
    - Level 0.
        
        ```jsx
        <Button onClick={showConfirm}>전송</Button>
        {isShowConfirm && (
        	<Confirm && onClick={() => {showMessage('성공')}}/>
        )}
        ```
        
    - Level 1.
        - `<Button>` 컴포넌트와 `confirm 활성화 여부`를 `<ConfirmButton/>` 이라는 컴포넌트로 추상화
        
        ```jsx
        <ConfirmButton onConfirm={() => {showMessage('성공')}>전송</ConfirmButton>
        ```
        
    - Level 2.
        - `message 라는 prop 만 넘겨서` confirm 창에 원하는 메시지를 보여 주도록 더욱 간단하게 추상화
        
        ```jsx
        <ConfirmButton message="성공">전송</ConfirmButton>
        ```
        
    - Level 3.
        - 모든 기능을 `<ConfirmButton/>` 컴포넌트 아래에 추상화 시킴.
        
        ```jsx
        <ConfirmButton/>
        ```
        

### 추상화 수준이 섞여 있으면 코드 파악이 어려워요

![image](https://user-images.githubusercontent.com/53039583/202887041-6de9d638-4682-4d61-b195-d3974528ee52.png)


- 구체적으로 작성되어 있는 노란색 컴포넌트를 보고 초록색 컴포넌트도 충분히 구체적으로 작성되어 있다고 짐작하게 될 수도 있다. 
내부를 보면 엄청나게 복잡한 코드가 숨겨져 있을 수도 있는데 말이다.
- 이렇게 되면, 코드를 읽는데 사고를 널뛰게 된다. 전체적인 코드가 어느 수준으로 구체적으로 기술됐는지 파악할 수 없기 때문이다.

![image](https://user-images.githubusercontent.com/53039583/202887046-f02e50d7-0b22-4116-ae8f-537742351f9e.png)

- 추상화 단계를 비슷하게 정리해주면 물 흐르듯이 파악하기가 쉽다.
- 위의 예시는 높은 추상화 단계로 정리했지만 상항에 따라 낮은 추상화 단계로 정리해도 된다.

# 실무에 바로 적용하기 위한 액션 아이템
## 1. 담대하게 기존 코드를 수정하기

> *두려워하지 말고 기존 코드를 씹고 뜯고 맛보고 즐기자*
> 
- 구조 뜯기를 두려워 하면, 클린한 실무 코드를 유지할 수 없다.
- Pull Request 에 File changes 의 양을 줄이고 싶다면 Mother branch 를 분기하여 리팩토링한 PR을 추가로 만드는 방법을 사용하면 좋다.

## 2. 큰 그림 보는 연습하기

> *그 때는 맞고 지금은 틀릴 수도 있다. 기능 추가 자체는 클린해도 전체적으로는 어지러울 수 있다.*
> 
- 기존에 깨끗하던 코드에 내가 기능을 추가하면서 망쳐놓을 수도 있다.
- 그리고 `내 기능 추가 자체는 클린해도 큰 그림으로는 어지러운 코드일 수 있음`을 유의하자.

## 3. 팀과 함께 공감대 형성하기

> *코드에 정답은 없다. 명시적으로 이야기하는 시간이 필요하다.*
> 
- 코드 리뷰에서 클린 코드 관련 댓글을 달아도 될 지 고민하게 될 수도 있다.
- 당장은 사소한 이슈일지라도, `적극적으로 이야기를 해보는 습관`을 들이자.

## 4. 문서로 적어 보기

> *글로 적어야 명확해진다.*
> 
- `이 코드가 향후 어떤 점에서 위험할 수 있는지?`
- `어떻게 개선할 수 있는지?`