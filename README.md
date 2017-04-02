# auth
Example of authentication via local and Google OAuth2.

```dart
import 'package:angel_auth_google/angel_auth/google.dart';
import 'package:angel_common/angel_common.dart';
import 'package:googleapis/plus/v1.dart';

const Map<String, String> GOOGLE_AUTH_CONFIG = const {
  'id': '<client-id>',
  'secret': '<client-secret>',
  'redirect_uri': 'http://localhost:3000/auth/google/callback'
};

const List<String> GOOGLE_AUTH_SCOPES = const [
  PlusApi.PlusMeScope,
  PlusApi.UserinfoEmailScope,
  PlusApi.UserinfoProfileScope
];

GoogleAuthCallback googleAuthCallback(Service userService) {
  return (client, Person profile) async {
    List<User> users = await userService.index({
      'query': {'googleId': person.id}
    });

    if (users.isNotEmpty)
      return users.first;
    else {
      return await userService.create({'googleId': person.id});
    }
  };
}

LocalAuthVerifier localVerifier(Service userService) {
  return (String username, String password) async {
    List<User> users = await userService.index({
      'query': {'username': username}
    });

    if (users.isNotEmpty) {
      return users.firstWhere((user) {
        var hash = hashPassword(password, user.salt, app.jwt_secret);
        return user.username == username && user.password == hash;
      }, orElse: () => null);
    }
  };
}

configureAuth(Angel app) async {
  var auth = new AngelAuth()..strategies.addAll([
    new LocalAuthStrategy(localVerifier(app.service('api/users')),
    new GoogleStrategy(
      config: GOOGLE_AUTH_CONFIG,
      scopes: GOOGLE_AUTH_SCOPES,
      callback: googleAuthCallback(app.service('api/users')))
  ]);
  
  app.post('/auth/local', auth.authenticate('local'));
  app.get('/auth/google', auth.authenticate('google'));
  app.get(
    '/auth/google/callback',
      auth.authenticate('google', new AngelAuthOptions(callback: confirmPopupAuthentication()))
  );
}
```
