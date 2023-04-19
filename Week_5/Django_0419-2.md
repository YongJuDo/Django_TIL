# Django with Ajax

<br>

## 개요
- `비동기(Asynchronus)`
  - 작업을 시작한 후 결과를 기다리지 않고 다음 작업을 처리하는 것 (병렬적 수행)
  - 시간이 필요한 작업들은 요청을 보낸 뒤 응답이 빨리 오는 작업부터 처리
  - 예시
    - Gmail에서 메일 전송을 누르면 목록 화면으로 전환되지만 실제로 메일을 보내는 작업은 병렬적으로 뒤에서 처리됨

- `Ajax(Aynchronous JavaScript And XML)`
  - 비동기적인 웹 애플리케이션 개발을 위한 프로그래밍 기술명
  - 사용자의 요청에 대한 즉각적인 반응을 제공하면서, 페이지 전체를 다시 로드하지 않고 필요한 부분만 업데이트 하는 것을 목표

- `XMLHttpRequest`
  - JavaScript 객체로, 클라이언트와 서버 간에 데이터를 비동기적으로 주고 받을 수 있도록 해주는 객체
  - JavaScript 코드에서 서버에 요청을 보내고, 서버로부터 응답을 받을 수 있음


## 비동기 요청
- `Axios`
  - JavaScript에서 HTTP 요청을 보내는 라이브러리
  - 주로 프론트엔드 프레임워크에서 사용

- Axios 기본 문법
  - ```javascript
    <script src="https://cdn.jsdelivr.net/npm/axios/dit/axios.min.js"></script>
    <script>
      axios({
        method: 'HTTP 메서드',
        url: '요청 URL',
      })
        .then(성공하면 수행할 콜백함수)
        .catch(실패하면 수행할 콜백함수)
    </script>
    ```
    - get, post 등 여러 method 사용가능
    - then을 이용해서 성공하면 수행할 로직을 작성
    - catch를 이용해서 실패하면 수행할 로직을 작성



## 팔로우 with Ajax
- 팔로우 구현 - 1
  - axios CDN 작성
  - ```html
    <!-- accounts/profile.html -->
    <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
    <script>

    </script>
    </body>
    ```

- 팔로우 구현 - 2
  - form 요소 선택을 위해 id 속성 지정 및 선택
  - 불필요해진 action과 method 속성은 삭제
  - axios 요청 준비
  - ```html
    <!-- accounts/profile.html -->
    <form id="follow-form">
    ...
    </form>

    <script>
      const form = document.querySelector('#follow-form')

      form.addEventListener('sumbit', function (event){
        event.preventDefault()
        axios({
          method: 'post',
          url: `/accounts/${유저pk}/follow/`,
        })
      }) 
    </script>
    ```

- 팔로우 구현 - 3
  - 현재 axios로 POST 요청을 보내기 위해 필요한 것
    - 1 - url에 작성할 user pk는 어떻게 작성?
    - 2 - csrftoken은 어떻게 보낼까?

- 팔로우 구현 - 4
  - url에 작성할 user pk 가져오기 (HTML -> JavaScript)
  - ```html
    <!-- accounts/profile.html -->
    <form id="follow-form" data-user-id="{{ person.pk }}">
    ...
    </form>

    <script>
      const form = document.querySelector('#follow-form')

      form.addEventListener('submit', function (event){
        event.preventDefault()

        const userId = event.target.dataset.userId
        axios({
          method: 'post',
          url: `/accounts/${userId}/follow/`,
        })
      }) 
    </script>
    ```

- 'data-*' attributes
  - 사용자 지정 데이터 특성을 만들어 임의의 데이터를 HTML과 DOM사이에서 교환 할 수 있는 방법
  - data-test-value라는 이름의 특성을 지정했다면 JavaScript에서는 element.dataset.testValue로 접근할 수 있음

- 팔로우 구현 - 5
  - 요청 url 작성 마치기
  - ```html
    <!-- accounts/profile.html -->
    <script>
      const form = document.querySelector('#follow-form')

      form.addEventListener('submit', function (event){
        event.preventDefault()

        const userId = event.target.dataset.userId
        axios({
          method: 'post',
          url: `/accounts/${userId}/follow/`,
        })
      }) 
    </script>
    ```

- 팔로우 구현 - 6
  - 먼저 hidden 타입으로 숨어있는 csrf값을 가진 input 태그를 선택
  - ```html
    <!-- accounts/profile.html -->
    <script>
      const form = document.querySelector('#follow-form')
      const csrftoken = document.querySelector('[name=csrfmiddlewaretoken]').value
    </script>
    ```

- 팔로우 구현 - 7
  - AJAX로 csrftoken을 보내는 방법
  - ```html
    <!-- accounts/profile.html -->
    <script>
      const form = document.querySelector('#follow-form')
      const csrftoken = document.querySelector('[name=csrfmiddlewaretoken]').value

      form.addEventListener('submit', function (event){
        event.preventDefault()

        const = userId = event.target.dataset.userId

        axios({
          method: 'post',
          url: `/accounts/${userId}/follow/`,
          headers: {'X-CSRFToken': csrftoken,}
        })
      }) 
    </script>
    ```

- 팔로우 구현 - 7
  - 팔로우 버튼을 토글하기 위해서는 현재 팔로우가 된 상태인지 여부 확인 필요
  - axios 요청을 통해 받는 reponse 객체를 활용해 view 함수를 통해서 팔로우 관계 여부를 파악 할 수 있는 변수를 담아 JSON 타입으로 응답하기

- 팔로우 구현 - 8
  - 팔로우 관계를 확인하기 위한 is_followed 변수 작성 및 JSON 응답
  - ```python
    # accounts/views.py
    from django.http import JsonResponse

    @login_required
    def follow(request, user_pk):
        User = get_user_model()
        you = User.objects.get(pk=user_pk)
        me = request.user

        if you != me:
            if me in you.followers.all():
                you.followers.remove(me)
                is_followed = False
            else:
                you.followers.add(me)
                is_followed = True
            context = {
                'is_followed': is_followed,
            }
            return JsonResponse(context)
        return redirect('accounts:profile', you.username)
    ```

- 팔로우 구현 - 9
  - view 함수에서 응답한 is_followed를 사용해 버튼 토글하기
  - ```html
    <!-- accounts/profile.html -->
    <script>
      ...
      axios({
        method: 'post',
        url: `/accounts/${userId}/follow/`,
        headers: {'X-CSRFToken': csrftoken,}
      })
        .then((response) => {
          const isFollowed = response.data.is_followed
          const followBtn = document.querySelector('#follow-form > input[type=sumibt]')
          if (isFollowed === true) {
            followBtn.value = '언팔로우'
          } else {
            followBtn.value = '팔로우'
          }
        })
    </script>
    ```

- 팔로잉 & 팔로워 수 비동기 적용 - 1
  - 해당 요소를 선택할 수 있도록 span 태그와 id 속성 작성
  - ```html
    <!-- accounts/profile.html -->
    <div>
      팔로잉 : <span id="followings-count">{{ person.followings.all|length }}</span>/
      팔로워 : <span id="followers-count">{{ person.followers.all|length }}</span>
    </div>
    ```

- 팔로잉 & 팔로워 수 비동기 적용 - 2
  - 직전에 작성한 span 태그를 각각 선택
  - ```html
    <!-- accounts/profile.html -->
    <script>
    ...
    .then((response) => {
      const isFollowed = response.data.is_followed
      const followBtn = document.querySelector('#follow-form > input[type=sumibt]')
      if (isFollowed === true) {
        followBtn.value = '언팔로우'
      } else {
        followBtn.value = '팔로우'
      }
      const followingCountTag = document.querySelector('#followings-count')
      const followersCountTag = document.querySelector('#followers-count')
    })
    </script>
    ```

- 팔로잉 & 팔로워 수 비동기 적용 - 3
  - 팔로잉, 팔로워 인원 수 연산은 view 함수에서 진행하여 결과를 응답으로 전달
  - ```python
    # accounts/views.py
    @login_required
    def follow(request, user_pk):
      ...
      context = {
        'is_followed': is_followed,
        'followings_count': you.followings.count(),
        'followers_count': you.followers.count(),
      }
    ```

- 팔로잉 & 팔로워 수 비동기 적용 - 4
  - view 함수에서 계산된 결과를 응답에서 찾아 적용
  - ```html
    <!-- accounts/profile.html -->
    <script>
    ...

      .then((response) => {
      ...

        const followingsCountTag = document.querySelector('#followings-count')
        const followersCountTag = document.querySelector('#followers-count')
        const followingsCountData = response.data.followings_count
        const followersCountData = response.data.followers_count
        followingCountTag.textContent = followingsCountData
        followersCountTag.textContent = followersCountData
      })
    </script>
    ```


## 좋아요 with Ajax
- 좋아요 비동기 적용
  - 팔로우와 동일한 흐름 + forEach() + querySelectorAll()
  - index 페이지 각 게시글에 각각 좋아요 버튼이 있기 때문
  - 반복 + 목록 선택


## 참고
- 비동기를 사용하는 이유
  - 아주 큰 데이터를 불러온 뒤 실행하는 앱이 있을때, 동기로 처리한다면 데이터를 모두 불러온 뒤에야 앱의 실행 로직이 수행되므로 사용자들은 마치 앱이 멈춘 것과 같은 경험을 겪게 됨
  - 동기식 처리는 특정 로직이 실행되는 동안 다른 로직을 차단하기 때문에 마치 프로그램이 응답하지 않은 듯한 사용자 경험을 만들게 됨
  - 비동기로 처리한다면 먼처 처리되는 부분부터 보여줄수 있으므로, 사용자 경험에 긍정적인 효과를 볼 수있음. 이와 같은 이유로 많은 웹 기능은 비동기 로직을 사용해서 구현되어 있음