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

#### Future<String> HTTP ASYNC
```dart
import 'package:http/http.dart' as http;

class DataService{
  Future<String> makeRequestToApi() async {
    //https://demo-v3-laravelapi.gzeinnumer.com/api/products/all
    final uri = Uri.https("demo-v3-laravelapi.gzeinnumer.com", "/api/products/all");
    final response = await http.get(uri);
    return response.body;
  }
}
```
```dart
final _dataService = DataService();
var _response;

void _makeRequest() async {
    String res = await _dataService.makeRequestToApi();
    setState(() {
      _response = res;
    });
}
```

#### Column
```dart
Column(
    crossAxisAlignment: CrossAxisAlignment.start, //kiri - kanan
    mainAxisAlignment: MainAxisAlignment.center, //atas - bawah
    children: [
    ],
)
```
```dart
Row(
    crossAxisAlignment: CrossAxisAlignment.start, //atas - bawah
    mainAxisAlignment: MainAxisAlignment.center, //kiri - kanan
    children: [
    ],
)
```
#### Match Parent
```
SizedBox(
    width: double.infinity, //match parent
    child: Text('Login'),
),
```

#### Refactor Value
```dart
class WeatherResponse{
    final WeatherInfo? weatherInfo;

    String get iconUrl{
       return "https://openweathermap.org/img/wn/${weatherInfo!.icon}@2x.png";
    }
}
```

#### Json Converter
```dart
final tempInfoJson = json['main'];
final tempInfo = TemperatureInfo.fromJson(tempInfoJson);
```
```dart
class TemperatureInfo{
  final double? temperature;

  TemperatureInfo({this.temperature});

  factory TemperatureInfo.fromJson(Map<String, dynamic> json){
    final temperature = json['temp'];
    return TemperatureInfo(temperature: temperature);
  }
}
```

#### LisView Flutter
```
return ListView.builder(itemBuilder: (context, index) {
  return Card(
    child: ListTile(
      title: Text(res[index].title.toString()),
    ),
  );
});
```

#### Cubit
```dart
BlocProvider<PostCubit>(create: (context) => PostCubit()..getPost())
```
```dart
class PostCubit extends Cubit<List<Post>> {
  final _dataService = DataService();

  PostCubit() : super([]);

  void getPost() async {
    return emit(await _dataService.getPosts());
  }
}
```
```
body: BlocBuilder<PostCubit, List<Post>>(builder: (context, res) {
    if (res.isEmpty) {
      return const Center(child: CircularProgressIndicator());
    }
    return ListView.builder(
        itemCount: res.length,
        itemBuilder: (context, index) {
          return Card(
            child: ListTile(
              title: Text(res[index].title.toString()),
            ),
          );
        });
  }),
```

#### Exception
```
throw Exception('failed log in');
```

#### Use Again Bloc add
```
BlocProvider.of<PostBloc>(context).add(PullToRefreshEvent())
```
```
context.read<PostBloc>().add(PullToRefreshEvent())
```

#### Cubit To Bloc - List
- Provider
  - Cubit
```dart
return MaterialApp(
  home: MultiBlocProvider(
    providers: [
      BlocProvider<PostCubit>(create: (context) => PostCubit()..getPost())
    ],
    child: PostView(),
  ),
);
```
  - Bloc
```dart
return MaterialApp(
  home: MultiBlocProvider(
    providers: [
      BlocProvider<PostBloc>(create: (context) => PostBloc()..add(LoadPostEvent()))
    ],
    child: PostView(),
  ),
);
```

- State
  - Cubit
```dart
class PostCubit extends Cubit<List<Post>> {
  final _dataService = DataService();

  PostCubit() : super([]);

  void getPost() async {
    return emit(await _dataService.getPosts());
  }
}
```
  - Bloc
```dart
//event
abstract class PostEvent{}

class LoadPostEvent extends PostEvent{}

class PullToRefreshEvent extends PostEvent{}

//state
abstract class PostState{}

class LoadingPostState extends PostState{}

class LoadedPostState extends PostState{
  final List<Post> list;

  LoadedPostState(this.list);
}

class FailedToLoadPostState extends PostState{
  final Error exception;

  FailedToLoadPostState(this.exception);
}

//bloc
class PostBloc extends Bloc<PostEvent, PostState>{
  final _dataService = DataService();

  PostBloc() : super(LoadingPostState());

  @override
  Stream<PostState> mapEventToState(PostEvent event) async* {
    if(event is LoadPostEvent || event is PullToRefreshEvent){
      yield LoadingPostState();
      try{
        final res = await _dataService.getPosts();
        yield LoadedPostState(res);
      } on Error catch(e){
        yield FailedToLoadPostState(e);
      }
    }
  }
}

```
- Builder
  - Cubit
```dart
body: BlocBuilder<PostCubit, List<Post>>(builder: (context, res) {
    if (res.isEmpty) {
      return const Center(child: CircularProgressIndicator());
    }
    return ListView.builder(
        itemCount: res.length,
        itemBuilder: (context, index) {
          return Card(
            child: ListTile(
              title: Text(res[index].title.toString()),
            ),
          );
        });
  }),
```
  - Bloc
```dart
body: BlocBuilder<PostBloc, PostState>(builder: (context, state) {
    if (state is LoadingPostState) {
      return const Center(child: CircularProgressIndicator());
    } else if (state is LoadedPostState) {
      return RefreshIndicator(
        onRefresh: () async {
          return BlocProvider.of<PostBloc>(context).add(PullToRefreshEvent());
        },
        child: ListView.builder(...),
      );
    } else if (state is FailedToLoadPostState) {
      return Center(
        child: Text('Error occured: ${state.exception.toString()}'),
      );
    } else {
      return Container();
    }
}),
```


---

```
Copyright 2022 M. Fadli Zein
```