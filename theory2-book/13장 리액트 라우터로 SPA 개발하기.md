```js

1. SPA 란?

- 한 개의 페이지로 이루어진 애플리케이션.
- 기존 전통적인 방식의 서버사이드 렌더링의 일을 브라우저가 담당하도록 한다.
- 싱글 페이지라고 해서 화면이 한 종류는 아니다. 예를 들어 블로그 개발을 한다면 홈, 포스트 목록, 포스트, 글쓰기 등의 화면이 있다.
- SPA의 경우 서버에서 사용자에게 제공하는 페이지는 한 종류이지만 해당 페이지에서 로딩된 자바스크립트와 현재 사용자 브라우저의 주소 상태에 따라 다양한 화면을 보여줄 수 있다.
- 다른 주소에 다른 화면을 보여주는 것을 라우팅이라 한다.
- 리액트 라우터 라이브러리는 클라이언트 사이드에서 이루어지는 라우팅을 아주 간단하게 구현할 수 있게 해주고,
- 더 나아가서 나중에 서버 사이드 렌더링을 할 때도 라우팅을 도와주는 컴포넌트들을 제공해준다.

* 서버에서는 index.html 파일 단 하나만 렌더링 한다. 리액트는 자바스크립트 파일을 바탕으로 index.html을 가공하여 많은 페이지를 만들어내는 것임

* SPA 의 단점

1) 웹앱의 규모가 커지면 JS 파일이 너무 커진다. 페이지 로딩 시 사용자가 실제로 방문하지 않을 수도 있는 페이지의 스크립트도 불러오기 때문. (index.html에 모든 JS 파일 때려박기 때문에 index.html을 로딩하는 순간 모든 JS 파일이 불러와짐) 하지만 코드 스플리팅을 사용하면 라우트별로 파일들을 나누어서 트래픽과 로딩 속도를 개선할 수 있다.

2) 검색엔진 최적화 SEO 문제가 발생함.
- 리액트를 통한 서버사이드 렌더링으로 해결 가능







2. 프로젝트 시작하기

- npm i react-router-dom
- import { BrowserRouter } from "react-router-dom";

1) 프로젝트에 라우터 적용 : BrowserRouter라는 컴포넌트로 최상위 컴포넌트를 감싼다. 이 컴포넌트는 웹 애플리케이션에 HTML5 history api를 사용하여 페이지를 새로고침하지 않고도 주소를 변경하고, 현재 주소에 관련된 정보를 props로 쉽게 조회하거나 사용할 수 있도록 해준다.

ReactDOM.render(
    <React.StrictMode>
        <BrowserRouter>
            <RouterExam />
        </BrowserRouter>
    </React.StrictMode>,
    document.getElementById("root"),
);





2) Route 컴포넌트로 특정 주소에 컴포넌트 연결

import { Route, Link } from "react-router-dom";

<Route path="주소규칙" component={보여줄 컴포넌트} />

const RouterExam = () => {
    return (
        <div>
            --> 두 개의 라우트컴포넌트 중에서 URL 에 맞는 라우트컴포넌트가 렌더링된다.
            <Route path="/" component={RouterHome} exact={true}></Route>
            <Route path="/about" component={RouterAbout}></Route>
        </div>
    );
};




3) Link 컴포넌트를 사용하여 다른 주소로 이동하기

import { Route, Link } from "react-router-dom";

<Link to="주소">내용</Link>

- Link 컴포넌트는 클릭하면 다른 주소로 이동시켜 주는 컴포넌트. 일반적인 웹애플리케이션에서는 a 태그를 사용하여 페이지를 전환하는데 리액트 라우터를 사용할 때는 이 태그를 직접사용하면 안된다. a 태그는 페이지를 전환하는 과정에서 페이지를 새로 불러오기 때문에 애플리케이션이 들고 있던 상태들을 모두 날려버린다. 즉 렌더링 된 컴포넌트들도 모두 사라지고 처음부터 다시 렌더링 된다.

- Link 컴포넌트를 사용하여 페이지를 전환하면 페이지를 새로 불러오지 않고 애플리케이션은 그대로 유지한 상태에서 HTML5 history API 를 사용하여 페이지의 주소만 변경해준다. Link 컴포넌트 자체는 a태그로 이루어져 있지만, 페이지 전환을 방지하는 기능이 내장되어 있음.





4) Route 하나에 여러 개의 path 설정하기

- 리액트 라우터 v5부터 적용된 기능.

<Route path={["/about", "/info"]} component={RouterAbout}></Route>
-> path props를 배열로 설정해 주면 여러 경로에서 같은 컴포넌트를 보여 줄 수 있다.





5) URL 파라미터

- 페이지 주소를 정의할 때 가끔은 유동적인 값을 전달해야 할 때도 있다. 이는 파라미터와 쿼리로 나눌 수 있는데, 일반적으로 파라미터는 특정 아이디 혹은 이름을 사용하여 조회할 때 사용하고 쿼리는 어떤 키워드를 검색하거나 페이지에 필요한 옵션을 전달할 때 사용한다.

파라미터 : /profile/jinho
쿼리 : /about?details=true

** URL 파라미터를 사용할 때는 라우트로 사용되는 컴포넌트에서 받아 오는 match라는 객체 안의 params 값을 참조한다. match 객체 안에는 현재 컴포넌트가 어떤 경로 규칙에 의해 보이는지에 대한 정보가 들어있다.

ex)

const data = {
    jinho: {
        name: "park",
        desc: "개발자",
    },
    gildong: {
        name: "hong",
        desc: "주인공",
    },
};

const RouterExam = () => {
    return (
        <div>
            <ul>
                <li>
                    <Link to="/">홈</Link>
                </li>
                <li>
                    <Link to="/about">소개</Link>
                </li>
                <li>
                    <Link to="/profile/jinho">jinho profile</Link>
                </li>
                <li>
                    <Link to="/profile/gildong">gildong profile</Link>
                </li>
            </ul>
            <Route path="/" component={RouterHome} exact={true}></Route>
            <Route path={["/about", "/info"]} component={RouterAbout}></Route>
            <Route path="/profile/:username" component={Profile}></Route> --> 여기서 match 객체를 Profile컴포넌트의 props로 전달해준다.
        </div>
    );
};

 const Profile = ({ match }) => {
    const { username } = match.params;
    const profile = data[username];
    if (!profile) {
        return <div>존재하지 않는 사용자입니다. </div>;
    }
    return (
        <div>
            <h3>
                {username}({profile.name})
            </h3>
            <p>{profile.desc}</p>
        </div>
    );
};






6) URL 쿼리

- 쿼리는 location 객체에 들어 있는 search 값에서 조회할 수 있다. location 객체는 라우트로 사용된 컴포넌트에게 props로 전달되며 웹 애플리케이션의 현재 주소에 대한 정보를 지니고 있다.

http://localhost:3000/about?detail=true
주소로 들어갔을 때의 location 객체 값
=> {
    "pathname" : "/about",
    "search" : "?detail=true", --> 실질적인 쿼리스트링. location.search
    "hash": ""
}

URL 쿼리를 읽을 때는 위 객체가 지닌 값 중에서 search값을 확인해야 한다. 이 값은 문자열 형태로 되어 있기 때문에 search 값에서 특정 값을 읽어 오기 위해서는 이 문자열을 객체 형태로 변환해주어야 한다. qs 라이브러리 이용

import qs from "qs";

const RouterAbout = ({ location }) => {
    const query = qs.parse(location.search, {
        ignoreQueryPrefix: true, --> 문자열의 맨 앞의 ?를 생략한다.
    });


->> 이를 통해
{
    "pathname" : "/about",
    "search" : "?detail=true",
    "hash": ""
} 이것이
{
    pathname : "/about",
    search : "?detail=true",
    hash: ""
} 이렇게 된다.


    const showDetail = query.detail === "true"; --> 쿼리의 파싱 결과 값은 문자열이다.
    return (
        <div>
            <h1>소개페이지</h1>
            <p>소개입니다.</p>
            {showDetail && <p>detail 값이 true입니다.</p>}
        </div>
    );
};

** 쿼리를 사용할 때는 쿼리 문자열을 객체로 파싱하는 과정에서 결과 값은 언제나 문자열이라는 점에 주의해야함. 그렇기 때문에 숫자를 받아와야하면 parseInt를 사용해야하고 논리 자료형 값을 사용해야하는 경우는 "true" or "false"같은 문자열 값을 그대로 사용해야한다.






7) 서브 라우트

- 서브 라우트는 라우트 내부에 또 라우트를 정의하는 것을 의미. 그냥 라우트로 사용되고 있는 컴포넌트의 내부에 Route 컴포넌트를 또 사용하면 된다.

ex)

const Profiles = () => {
    return (
        <div>
            <ul>
                <li>
                    <Link to="/profiles/jinho">jinho</Link>
                </li>
                <li>
                    <Link to="/profiles/gildong">gildong</Link>
                </li>
            </ul>
            <Route
                path="/profiles" --> 그냥 profiles일 때
                exact
                render={() => <div>사용자를 선택해주세요</div>}
            ></Route>
            <Route path="/profiles/:username" component={Profile}></Route>
            --> /profiles/:username일 때
        </div>
    );
};

const Profile = ({ match }) => {
    const { username } = match.params;
    const profile = data[username];
    if (!profile) {
        return <div>존재하지 않는 사용자입니다. </div>;
    }
    return (
        <div>
            <h3>
                {username}({profile.name})
            </h3>
            <p>{profile.desc}</p>
        </div>
    );
};

const RouterExam = () => {
    return (
        <div>
            <ul>
                <li>
                    <Link to="/">홈</Link>
                </li>
                <li>
                    <Link to="/about">소개</Link>
                </li>
                <li>
                    <Link to="/profiles">profile</Link>
                </li>
            </ul>
            <Route path="/" component={RouterHome} exact={true}></Route>
            <Route path={["/about", "/info"]} component={RouterAbout}></Route>
            <Route path="/profiles" component={Profiles}></Route>
        </div>
    );
};





8) 리액트 라우터 부가 기능



* history

- hisrory 객체는 라우트로 사용된 컴포넌트에 match, location과 함께 전달되는 props 중 하나로, 이 객체를 통해 컴포넌트 내에 구현하는 메서드에서 라우터 API를 호출할 수 있다. 예를 들어 특정 버튼을 눌렀을 때 뒤로 가거나, 로그인 후 화면을 전환하거나 다른 페이지로 이탈하는 것을 방지해야할 때 history를 활용한다.

ex)

<Link to="/history">history</Link>

<Route path="/history" component={historySample}></Route>

const historySample = ({ history }) => {
    const handleGoBack = () => {
        history.goBack(); --> 뒤로가기
    };

    const handleGohome = () => {
        history.push("/"); --> 설정한 url로 이동하기
    };
    return (
        <div>
            <button onClick={handleGoBack}>뒤로 가기</button>
            <button onClick={handleGohome}>홈으로 가기</button>
        </div>
    );
};

- history.block, unblock도 있는데, useEffect와 함께 써야하는 듯함.




* withRouter

- withRouter함수는 HOC이다. 라우트로 사용된 컴포넌트가 아니어도 match, location, history 객체에 접근할 수 있게 해준다. React.memo와 사용방법 같음.
- 제대로 된 결과값을 내려면 부모 자식 관계를 잘 정의해야함.

const WithRouterSample = withRouter(({ match, location, history }) => {
    return (
        <div>
            <h4>location</h4>
            <textarea
                value={JSON.stringify(location, null, 2)}

                -> null, 2를 각각 넣어주면 JSON에 들여쓰기가 적용된 상태로 문자열이 만들어진다.

                rows="7"
            ></textarea>
            <h4>match</h4>
            <textarea
                value={JSON.stringify(match, null, 2)}
                rows="7"
            ></textarea>
            <button
                onClick={() => {
                    history.push("/");
                }}
            >
                홈으로 가버리기
            </button>
        </div>
    );
});

-> 이렇게 써주면 됨
<WithRouterSample></WithRouterSample> -> profile컴포넌트의 param이용하고 싶으면 profile컴포넌트 렌더링 부분에 작성해주면 된다.





* switch

- 스위치 컴포넌트는 여러 route를 감싸서 그 중 일치하는 단 하나의 라우트만을 렌더링 시켜준다.
스위치를 사용하면 모든 규칙과 일치하지 않을 때 보여 줄 not found 페이지도 구현할 수 있다

import { Route, Link, withRouter, Switch } from "react-router-dom";

  <Switch>
      <Route path="/" component={RouterHome} exact={true}></Route>
      <Route
          path={["/about", "/info"]}
          component={RouterAbout}
      ></Route>
      <Route path="/profiles" component={Profiles}></Route>
      <Route path="/history" component={historySample}></Route>
      <Route render={({ location }) => (
                        <div>{location.pathname} not found</div>
                    )}></Route>
      --> ** path를 정의하지 않으면 모든 상황에 렌더링 된다.
      --> ** render 자체가 컴포넌트를 렌더링 하는 거임(익명 컴포넌트) 따라서 당연히 match, location, history 사용할 수 있다.
  </Switch>

==> 이런식으로 활용 가능


* NavLink

- NavLink는 Link와 비슷하다 현재 경로와 Link에서 사용하는 경로가 일치하는 경우 특정 스타일 혹은 CSS 클래스를 적용할 수 있는 컴포넌트이다.
- Link와 사용방법은 거의 비슷
- styled(NavLink)`` -> 이런식으로 사용 가능
- <NavLink to="/"> .... 이런식으로




```
