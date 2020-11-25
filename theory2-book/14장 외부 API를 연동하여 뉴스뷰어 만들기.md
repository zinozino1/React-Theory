```js

1. 비동기 작업의 이해

- 서버의 API 수신할 때 작업을 비동기로 처리해야함.
- 자바스크립트에서 비동기 작업을 할 때 가장 흔히 사용하는 방법은 콜백 함수를 사용하는 것


2. axios

React.js 는 효율적인 UI 구현을 위한 라이브러리. HTTP Client 를 내장하고있는 Angular 와는 다르게, React.js 는 따로 내장 클래스가 존재하지 않는다. 따라서 React.js 어플리케이션에서 AJAX 를 구현하려면 JavaScript 내장객체인 XMLRequest 를 사용하거나, 다른 HTTP Client 라이브러리와 함께 사용해야한다

axios는 node.js와 브라우저를 위한 http통신 javascript 라이브러리

모든 브라우저를 지원한다 (Fetch와 달리 크로스 브라우징에 최적화)

* 특징
- 브라우저에선 XMLHttpRequests을 nodejs에선 http 요청 생성
- Promise API 지원
- 요청과 응답을 중단
- JSON 데이터 자동 변환
- 요청 취소
- XSRF로부터 보호하기위한 클라이언트 측 지원

* 예제
GET 요청
// GET request for remote image
axios({
  method: 'get',
  url: 'http://bit.ly/2mTM3nY',
  responseType: 'stream'
})
  .then(function (response) {
    response.data.pipe(fs.createWriteStream('ada_lovelace.jpg'))
  });
or

axios.get('/user', {
    params: {
      ID: 12345
    }
  })
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  })
  .finally(function () {
    // always executed
  });

POST 요청
axios({
  method: 'post',
  url: '/user/12345',
  data: {
    firstName: 'Fred',
    lastName: 'Flintstone'
  }
});
or

axios.post('/user', {
    firstName: 'Fred',
    lastName: 'Flintstone'
  })
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });


예제 2)
const AxiosExam = () => {
    const [data, setData] = useState(null);
    const onClick = () => {
        console.log(0);
        axios
            .get("https://jsonplaceholder.typicode.com/todos/1")
            .then((res) => {
                console.log(1);
                setData(res.data);
            });
        console.log(2);
    };
    return (
        <div>
            <button onClick={onClick}>api호출</button>
            {data && <p>{JSON.stringify(data, null, 2)}</p>}
        </div>
    );
}; ----> 0, 2, 1 순으로 출력

const AxiosExam = () => {
    const [data, setData] = useState(null);
    const onClick = async () => {
        console.log(0);
        await axios
            .get("https://jsonplaceholder.typicode.com/todos/1")
            .then((res) => {
                console.log(1);
                setData(res.data);
            });
        console.log(2);
    };
    return (
        <div>
            <button onClick={onClick}>api호출</button>
            {data && <p>{JSON.stringify(data, null, 2)}</p>}
        </div>
    );
}; ----> 0, 1, 2 순으로 출력

뉴스api키 : 0e1636874c194cf7bed28aef2858db55
http://newsapi.org/v2/top-headlines?country=kr&apiKey=0e1636874c194cf7bed28aef2858db55

```
