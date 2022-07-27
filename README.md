# MyReminderFlutter

#### Padding
```dart
Padding(
    padding: const EdgeInsets.symmetric(horizontal: 40),
    child: Column(
        ...
    ),
),
```
- Left Right
```dart
EdgeInsets.symmetric(horizontal: 40)
```
- Top Bottom
```dart
EdgeInsets.symmetric(vertical: 40)
```

#### Repository

- [Source](https://pub.dev/documentation/flutter_bloc/latest/flutter_bloc/MultiRepositoryProvider-class.html)

- Single
```dart
return MaterialApp(
  home: RepositoryProvider(
    create: (context) => AuthRepository(),
    child: LoginView(),
  ),
);
```

- Multi
```dart
return MaterialApp(
  home: MultiRepositoryProvider(
    providers: [
      RepositoryProvider(create: (context) => AuthRepository())
    ],
    child: LoginView(),
  ),
);
```

#### Bloc

- [Source]()

- Single
```dart
return Scaffold(
  body: BlocProvider(
    create: (context) => LoginBloc(
      authRepo: context.read<AuthRepository>(),
    ),
    child: _loginForm(),
  ),
);
```

- Multi
```dart
return Scaffold(
  body: MultiBlocProvider(
    providers: [
      BlocProvider(
        create: (context) => LoginBloc(
          authRepo: context.read<AuthRepository>(),
        ),
      )
    ],
    child: _loginForm(),
  ),
);
```

#### CallBack
```dart
class BoardTile extends StatelessWidget {
  const BoardTile({
    Key? key,
    required this.tileDimension, this.onPressed,
  }) : super(key: key);

  final double tileDimension;
  final VoidCallback? onPressed;

  @override
  Widget build(BuildContext context) {
    return Container(
      width: tileDimension,
      height: tileDimension,
      child: FlatButton(
        onPressed: onPressed,
        child: Image.asset('images/x.png'),
      ),
    );
  }
}
```
```dart
BoardTile(tileDimension: tileDimension, onPressed: (){},),
```


#### CHUNK Make New List From List

```
[1,2,3,4,5,6]
to
[ [1,2,3] , [4,5,6] ]
```
```
import 'dart:math';

List<List<TileState>> chunk(List<TileState> list, int size) {
  return List.generate(
    (list.length / size).ceil(),
    (index) => list.sublist(
      index * size,
       min(index * size + size, list.length),
    ),
  );
}
```

---

```
Copyright 2022 M. Fadli Zein
```