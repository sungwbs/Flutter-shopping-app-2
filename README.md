# Flutter-shopping-app-2

- 개발환경: AndroidStudio / Firebase
- clone coding
<hr>
![](https://velog.velcdn.com/images/sungwbs/post/a3923b19-f098-403b-90ad-268a7cfb056f/image.png)


▶ **models Folder - auth.dart**
```c
import 'package:firebase_auth/firebase_auth.dart';
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

// 인증 상태를 나타내는 열거형
enum AuthStatus {
  registerSuccess, //회원가입 성공
  registerFail,
  loginSuccess,  //로그인 성공
  loginFail
}

// FirebaseAuth 인증을 제공하는 클래스 // ChangeNotifier 상태 변경을 알리는 클래스
class FirebaseAuthProvider with ChangeNotifier {
  FirebaseAuth authClient; // authClient 객체를 통해 다양한 사용자 인증 관련 메서드를 호출할 수 있습니다.
  User? user;  // user 변수는 값을 가질 수도 있고, null일 수도 있습니다.

  FirebaseAuthProvider({auth}) : authClient = auth ??  //인스턴스 초기화
      FirebaseAuth.instance; // 삼항연산자 사용. null이 아닌 경우에는 그 값을 사용하고, null인 경우에는 FirebaseAuth 클래스의 기본 인스턴스를 사용

  // 이메일로 로그인하는 메서드  // 25-이메일과 비밀번호를 사용하여 새 사용자를 등록합니다.
  Future<AuthStatus> registerWithEmail(String email, String password) async {
    try {
      UserCredential credential = await
      authClient.createUserWithEmailAndPassword(email: email, password: password);
      return AuthStatus.registerSuccess;
    } catch (e) {
      return AuthStatus.registerFail;
    }
  }
  // 34-등록된 사용자로 로그인합니다.
  Future<AuthStatus> loginWithEmail(String email, String password) async {
    try {
      await authClient.signInWithEmailAndPassword(email: email, password: password).then(
              (credential) async {
            user = credential.user;
            SharedPreferences prefs = await SharedPreferences.getInstance();
            prefs.setBool('isLogin', true);
            prefs.setString('email', email);
            prefs.setString('password', password);
          }
      );
      return AuthStatus.loginSuccess;
    } catch (e) {
      return AuthStatus.loginFail;
    }
  }

  // 로그아웃하는 메서드 // 56-현재 로그인된 사용자를 로그아웃합니다.
  Future<void> logout() async {
    SharedPreferences prefs = await SharedPreferences.getInstance();
    prefs.setBool('isLogin', false);
    prefs.setString('email', '');
    prefs.setString('password', '');
    user = null;
    await authClient.signOut();
  }
}

```

> - `FirebaseAuthProvider` 클래스는 ChangeNotifier를 상속하며, 애플리케이션의 상태 변경을 관리합니다.
- `authClient` 변수를 통해 Firebase Authentication 기능에 접근할 수 있습니다.
- `registerWithEmail` 메서드는 이메일과 비밀번호를 사용하여 새로운 사용자를 등록합니다. 등록이 성공하면 `AuthStatus.registerSuccess`를 반환하고, 실패하면 `AuthStatus.registerFail`를 반환합니다.
- `loginWithEmail` 메서드는 이메일과 비밀번호로 사용자를 로그인합니다. 로그인이 성공하면 `AuthStatus.loginSuccess`를 반환하고, 실패하면 `AuthStatus.loginFail`를 반환합니다.
- `logout` 메서드는 현재 로그인된 사용자를 로그아웃합니다.
- 이러한 메서드를 통해 사용자의 인증 상태를 관리하고, 로그인, 회원가입, 로그아웃 기능을 제공합니다.

**FirebaseAuthProvider**
- 목적: Firebase Authentication을 사용하여 사용자 인증을 관리하는 클래스
주요 기능:
사용자 등록(registerWithEmail): 이메일과 비밀번호를 사용하여 새로운 사용자를 등록합니다. 등록 성공 또는 실패에 따라 상태를 반환합니다.
사용자 로그인(loginWithEmail): 이메일과 비밀번호로 사용자를 로그인합니다. 로그인 성공 또는 실패에 따라 상태를 반환합니다.
사용자 로그아웃(logout): 현재 로그인된 사용자를 로그아웃합니다.


**registerWithEmail(String email, String password):
**
- 목적: 새로운 사용자를 이메일과 비밀번호로 등록합니다.
매개변수:
email: 사용자 이메일
password: 사용자 비밀번호
반환값: 등록 성공 시 AuthStatus.registerSuccess, 실패 시 AuthStatus.registerFail

**loginWithEmail(String email, String password):**
- 목적: 등록된 사용자로 로그인합니다.
매개변수:
email: 사용자 이메일
password: 사용자 비밀번호
반환값: 로그인 성공 시 AuthStatus.loginSuccess, 실패 시 AuthStatus.loginFail

**logout():**
- 목적: 현재 로그인된 사용자를 로그아웃합니다.
반환값: 없음. 로그아웃 후 필요한 데이터를 업데이트합니다.


▶ **models Folder - login.dart**
```c
import 'package:flutter/material.dart';

 // 로그인 화면 상태를 관리하는 모델 클래스
class LoginModel extends ChangeNotifier {
  String email = "";
  String password = "";

  // 이메일 값을 설정하는 메서드
  void setEmail(String email) {
    this.email = email; // 계속 만들어지는 현재 객체를 지칭
    notifyListeners();  //리스너 알림
  }

  // 비밀번호 값을 설정하는 메서드
  void setPassword(String password) {
    this.password = password;
    notifyListeners();
  }
}
```
▶ **models Folder - register.dart**
```c
import 'package:flutter/material.dart';

// 회원가입 화면 상태를 관리하는 모델 클래스
class RegisterModel extends ChangeNotifier {
  String email = "";
  String password = "";
  String passwordConfirm = "";

  // 이메일 값을 설정하는 메서드
  void setEmail(String email) {
    this.email = email; //상위 클래스에 선언 된 변수를 가져옴
    notifyListeners();
  }

  // 비밀번호 값을 설정하는 메서드
  void setPassword(String password) {
    this.password = password;
    notifyListeners();
  }

  // 비밀번호 확인 값을 설정하는 메서드
  void setPasswordConfirm(String passwordConfirm) {
    this.passwordConfirm = passwordConfirm;
    notifyListeners();
  }
}
```
> - 그 외에 login.dart와 register.dart에 작성 된 코드를 통하여 각 로그인과 회원가입에 들어오는 매개변수 상태 값을 등록하여 여러 사용자의 정보를 업로드 해줍니다. 그리고 계속적으로 사용자 정보를 파이어베이스와 연동하여 설정하여 통신을 확인합니다.

![](https://velog.velcdn.com/images/sungwbs/post/b0018a7b-6426-4cd8-9694-b1fcf6e79a15/image.PNG)


▶ **왜? 파이어베이스를 사용했는 가?** 파이어베이스를 사용한 계기는 클라우드 데이터베이스를 무료로 이용가능하며, 일부의 금액 지불로 서버의 역할 기능 또한 가능한 서비스를 제공하고 있기에 따로 서버 구현하는 보수비용과 시간을 절약가능하다. 그러므로 1인 개발을 통한 경험을 쌓기에는 충분한 결과를 만들어 낼 수 있다. 

▶ **부족한 점** 
단점으로는 네이티브 앱에 메모리가 커서 그런지 중간마다 테스트를 하기 위해 실행을 하면, 램 사용량이 너무 많아져서 테스트 하기 힘들다. 또한 플러터 안에서 구현하려다 보니 확실히 기능면에서는 네이티브 앱 개발 방식하고 차이가 생길 수 밖에 없다. 

▶ ** 느낀점 및 피드백 **
객체지향언어를 사용하여 매서드를 설정하니, 여러 사용자들로 부터 데이터를 받아오거나 저장하는 것에 큰 장점을 보여주고 있고, 비동기적으로 코드가 돌아가니 코드가 대기하지 않는 장점도 있다. 단점을 보안하기 위해서는 다양한 라이브러리 및 컴파일시 실행 시간과 메모리사용량에 대해서 테스트를 통해 알아보고자한다. 마지막으로 dart언어를 플러터 프레임워크에서 사용하다보니 확실히 ios하고 Android 크로스플랫폼을 구성 할 수 있어서 좋은 거 같다.  

참고문헌
https://cholol.tistory.com/570
