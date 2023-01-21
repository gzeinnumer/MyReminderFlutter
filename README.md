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

#
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

#
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

#
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

#
#### CHUNK Make New List From List

```dart
[1,2,3,4,5,6]
to
[ [1,2,3] , [4,5,6] ]
```
```dart
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

#
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

#
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

#
#### Match Parent
```dart
SizedBox(
    width: double.infinity, //match parent
    child: Text('Login'),
),
```

#
#### Refactor Value
```dart
class WeatherResponse{
    final WeatherInfo? weatherInfo;

    String get iconUrl{
       return "https://openweathermap.org/img/wn/${weatherInfo!.icon}@2x.png";
    }
}
```

#
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

#
#### LisView Flutter
```dart
return ListView.builder(itemBuilder: (context, index) {
  return Card(
    child: ListTile(
      title: Text(res[index].title.toString()),
    ),
  );
});
```

#
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

#
#### Exception
```dart
throw Exception('failed log in');
```

#
#### Use Again Bloc add
```dart
BlocProvider.of<PostBloc>(context).add(PullToRefreshEvent())
```
```dart
context.read<PostBloc>().add(PullToRefreshEvent())
```

#
#### Cubit To Bloc - List
- Provider

Cubit
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
Bloc
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

Cubit
```dart
class PostCubit extends Cubit<List<Post>> {
  final _dataService = DataService();

  PostCubit() : super([]);

  void getPost() async {
    return emit(await _dataService.getPosts());
  }
}
```
Bloc
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

Cubit
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
Bloc
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

#
#### State Bloc Style

- Type 1
```dart
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
```
```dart
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

- Type 2 (like) Base Resources
```dart
class PostState {
  final List<Object> list;
  final ExampleStatus formStatus;

  PostState({
    this.list = const [],
    this.formStatus = const ExampleInit(),
  });

  PostState copyWith({
    List<Object>? list,
    ExampleStatus? formStatus,
  }) {
    return PostState(
      list: list ?? this.list,
      formStatus: formStatus ?? this.formStatus,
    );
  }
}
```
```dart
class ExampleBloc extends Bloc<ExampleEvent, PostState> {

  ExampleBloc() : super(PostState());

  @override
  Stream<PostState> mapEventToState(ExampleEvent event) async* {
    if (event is ExampleChanged) {
      yield state.copyWith(list: event.list);
    } else if (event is ExampleSubmitted) {
      yield state.copyWith(formStatus: ExampleOnShowLoading());
      try {
        yield state.copyWith(formStatus: ExampleOnSuccess());
      } on Exception catch (e) {
        yield state.copyWith(formStatus: ExampleOnFailed(e));
      }
    }
  }
}
```

#
#### Flutter Dialog Disable Outside
```dart
showDialog(
  barrierDismissible: false,
  builder: ...
);
```

#
#### FLutter CallBack
```dart
final Function()? onPositivePressed;
final ValueChanged<String>? onChanged;
```

#
#### Flutter Container Radius
```dart
Container(
    width: match_parent,
    decoration : BoxDecoration(
        color: Colors.white,
        borderRadius: new BorderRadius.circular(def_margin)
    ),
    margin: const EdgeInsets.all(def_margin),
    child: PaddingAll(
        child: Column(
            children: [
                ...
            ],
        ),
    ),
),
```

#
#### Flutter Layout Weight
```dart
return Scaffold(
  appBar: EmptyAppBar(),
  body: Stack(
    children: [
      BackgroundType4(),
      Container(
        child: Column(
          children: [
            Expanded(child: Container(color: Colors.red,)),
            Expanded(child: Container(color: Colors.blue,)),
          ],
        ),
      ),
    ],
  ),
);
```

#
#### Flutter Row Center
```dart
Row(
    mainAxisAlignment: MainAxisAlignment.spaceEvenly,
    children: []
)
```

#
#### Flutter List
```dart
List<T> datas = List<T>.empty(growable: true);
json['datas'].forEach((v) {
    datas.add(create(v));
});
```
```dart
final list = <Post>[];
```

#
#### Flutter Items Generator
```dart
final List<MyModel>? product_desc;

List<Widget> get items => itemGenerator(product_desc!);

List<Widget> itemGenerator(List<MyModel> data){
    List<Widget> d = List<Widget>.empty(growable: true);
    for(var i=0; i<data.length; i++){
      d.add(Text(data[i].description.toString()));
    }
    return d;
}
```
```dart
List<String> list = ['one', 'two', 'three', 'four'];
List<Widget> widgets = list.map((name) => new Text(name)).toList();
```
```dart
var list = ["one", "two", "three", "four"];

child: Column(
    mainAxisAlignment: MainAxisAlignment.center,
    children: <Widget>[
        for(var item in list ) Text(item)
    ],
),
```
```
https://stackoverflow.com/questions/50441168/iterating-through-a-list-to-render-multiple-widgets-in-flutter
```

#
#### Flutter Generate From Current Value
```dart
String get freeDeliveryString => freeDelivery(subTotal);

String freeDelivery(subTotal){
    if(subTotal >= 30){
        return "You have FREE Delivery";
    } else{
        double missing = 30.0-subTotal;
        return "Add \$${missing.toStringAsFixed(2)} for FREE Delivery";
    }
}
```

#
#### Flutter Generate Widget Array
```dart
var list = ["one", "two", "three", "four"];

@override
Widget build(BuildContext context) {
    return new MaterialApp(
        home: new Scaffold(
            appBar: new AppBar(
                title: new Text('List Test'),
            ),
            body: new Center(
                child: new Column( // Or Row or whatever :),
                children: createChildrenTexts(),
            ),
        ),
    );
}

List<Text> createChildrenTexts() {
    /// Method 1
    List<Text> childrenTexts = List<Text>();
    for (String name in list) {
        childrenTexts.add(new Text(name, style: new TextStyle(color: Colors.red),));
    }
    return childrenTexts;

    /// Method 2
    return list.map((text) => Text(text, style: TextStyle(color: Colors.blue),)).toList();
}
```

#
#### Flutter ListView.builder
```dart
Container(
  child: ListView.builder(
    shrinkWrap: true,
    itemCount: items.length,
    itemBuilder: (BuildContext context, int index){
      return Container(
        child: Text(
          items[index]['property']
        ),
      );
    },
  ),
);
```

#
#### Flutter Statusbar Color

Update Flutter 2.0 (Recommended):
```dart
AppBar(
    backwardsCompatibility: true,
    systemOverlayStyle: SystemUiOverlayStyle(statusBarColor: Colors.white),
)
```
Only Android (more flexibility):
```dart
SystemChrome.setSystemUIOverlayStyle(SystemUiOverlayStyle(
    systemNavigationBarColor: Colors.blue, // navigation bar color
    statusBarColor: Colors.pink, // status bar color
));
```
Both iOS and Android:
```dart
AppBar(
    backgroundColor: Colors.red, // status bar color
    brightness: Brightness.light, // status bar brightness
)
```

#
#### Flutter StatusBarColor NavigationColor Recomended
```dart
SystemChrome.setSystemUIOverlayStyle(SystemUiOverlayStyle(
    statusBarColor: Colors.white,
    statusBarBrightness: Brightness.dark,
    systemNavigationBarColor: Colors.white,
    systemNavigationBarIconBrightness: Brightness.dark
));
```
```dart
AppBar(
    backwardsCompatibility: true,
    systemOverlayStyle: SystemUiOverlayStyle(
        statusBarColor: Colors.white,
        statusBarBrightness: Brightness.dark,
        systemNavigationBarColor: Colors.white,
        systemNavigationBarIconBrightness: Brightness.dark,
    ),
)
```

#
#### Flutter DebugBanner
```dart
MaterialApp(
  debugShowCheckedModeBanner: false,
)
```

#
#### Flutter Update
```
flutter channel master
flutter upgrade
```

#
#### Callback View FLutter
```dart
class UsersView extends StatelessWidget {
  final _users = ["Kyle", "Andriana", "Andrew"];

  final ValueChanged didSelector;

  UsersView({Key? key, required this.didSelector}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text("Users"),
      ),
      body: ListView.builder(
            ...
                onTap: () => didSelector(user),
            ...
          }),
    );
  }
}
```
```dart
class _MyAppState extends State<MyApp> {
  String? _selectedUser;

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Navigator(
        pages: [
          MaterialPage(
            child: UsersView(
              didSelector: (user) {
                setState(() {
                  _selectedUser = user;
                });
              },
            ),
          ),
          ...
        ],
        ...
    );
  }
}
```

#
#### Flutter List
```dart
List<String> restaurans = ["McDonald\'s", "Pizza Hut", "Besto", "Geprek"];
List<String> restaurans = [];
```

#
#### Flutter Chunk
```dart
[1,2,3,4,5,6]

[[1,2,3],[4,5,6]]
```
```dart
import 'dart:math';

enum TileState { EMPTY, CROSS, CIRCLE }

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
```dart
var _boardState = List.filled(9, TileState.EMPTY);
```
```dart
child: Column(
  children: chunk(_boardState, 3).asMap().entries.map((entry) {
    final chunkIndex = entry.key;
    final tileStateChunk = entry.value; //this is array //[[1,2,3]]

    return Row(
      children: tileStateChunk.asMap().entries.map((innerEnty) {
        final innerIndex = innerEnty.key;
        final tileState = innerEnty.value;
        final tileIndex = (chunkIndex * 3) + innerIndex; //[[1]]

        return Container();
      }).toList(),
    );
  }).toList(),
),
```

#
#### ToggleButton
```dart
List<bool> _selection = [true, false, false];
```
```dart
ToggleButtons(
  children: const [
    Text('10%'),
    Text('15%'),
    Text('20%'),
  ],
  isSelected: _selection,
  onPressed: updateSelection,
),
```
```dart
setState(() {
  for (int i = 0; i < _selection.length; i++) {
    _selection[i] = selectedIndex == i;
  }
});
```

#
#### Http
```dart
  Future<WeatherResponse> getWeather(String city) async {
    //https://api.openweathermap.org/data/2.5/weather?q=Yucaipa&appid=0b4be60a8797e131f49efc402fbbc0ed

    // final queryParameters = {"lat": "35", "lon":"139", "appid": "0b4be60a8797e131f49efc402fbbc0ed"};
    final queryParameters = {
      "q": city,
      "appid": "0b4be60a8797e131f49efc402fbbc0ed",
      "units": "imperial"
    };

    final uri = Uri.https(
        "api.openweathermap.org", "/data/2.5/weather", queryParameters);

    final response = await http.get(uri);

    print(response.body);

    final json = jsonDecode(response.body);
    return WeatherResponse.fromJson(json);
  }
```

#
#### Tabbar
```dart
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: DefaultTabController(
        length: 1,
        child: Scaffold(
          backgroundColor: Colors.white70,
          appBar: AppBar(
            bottom: const TabBar(
              tabs: [
                Text("List"),
              ],
            ),
          ),
          body: TabBarView(
            children: [
              Container(),
            ],
          ),
        ),
      ),
    );
  }
```

#
#### Flutter Add Bloc

type 1
```dart
  home: MultiBlocProvider(
    providers: [
      BlocProvider<PostBloc>(create: (context) => PostBloc())
    ],
    child: PostView(),
  ),
```
```dart
  body: Builder(
      builder: (context) {
        context.read<PostBloc>().add(PostInitEvent());
        return BlocBuilder<PostBloc, PostState>(builder: (context, state) {
          ...
          return Container();
        });
      },
  ),
```
type 2
```dart
  home: MultiBlocProvider(
    providers: [
      BlocProvider<PostBloc>(create: (context) => PostBloc()..add(PostInitEvent()))
    ],
    child: PostView(),
  ),
```

#
#### FLutter If Return View
```dart
  body: Builder(
      builder: (context) {
        //if
        //for
        //switch
        return Container();
      },
  ),
```

#
#### FLutter Future Multi
```dart
///async function
final response = await Future.wait([
  _pokemonRepository.getPokemonInfo(value),
  _pokemonRepository.getPokemonSpeciesInfo(value)
]);
final pokemonInfo = response[0] as PokemonInfoResponse;
final pokemonSpeciesInfo = response[1] as PokemonSpeciesInfoResponse;
```

#
#### Flutter Color
```dart
backgroundColor: const Color(0xFFF2F2F2)
```

#
#### Flutter Layout Weight
```dart
child: Column(
    children: [
      Expanded(
        flex: 0,
        child: Container(),
      ),
      Expanded(
        flex: 1,
        child: SizedBox(
          width: double.infinity,
          child: Container(),
        ),
      )
    ],
),
```

#
#### FLutter OnClick
```dart
return GestureDetector(
    onTap: () => BlocProvider.of<NavCubit>(context).showPokemonDetails(state.pokemonListing[index].id),
    child: Container(),
);
```

#
#### FLutter TextFormField
```dart
  Widget _passwordField() {
    return TextFormField(
      obscureText: true,
      decoration: const InputDecoration(
          icon: Icon(Icons.security), hintText: "Password"),
      validator: (value) => null,
    );
  }
```

#
#### Flutter copyWith
```dart
class LoginState {
  final String? username;
  final String? password;

  LoginState({this.username, this.password});

  LoginState copyWith({
    String? username,
    String? password,
  }) {
    return LoginState(
      username: username ?? this.username,
      password: password ?? this.password,
    );
  }
}
```

#
#### Flutter Future Delay
```dart
Future<void> login() async{
    Future.delayed(const Duration(seconds: 3));
}
```

#
#### Flutter Form Key
```dart
final _formKey = GlobalKey<FormState>();

Widget _content() {
    return Form(
        key: _formKey,
        child: Container(
          margin: const EdgeInsets.symmetric(horizontal: 16),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            crossAxisAlignment: CrossAxisAlignment.end,
            children: [
                TextFormField(
                  validator: (value) => null,
                )
            ],
          ),
        ),
    );
}
```
```dart
ElevatedButton(
  onPressed: () {
    if (_formKey.currentState!.validate()) {
      context.read<LoginBloc>().add(LoginSubmittedEvent());
    }
  },
  child: const Text('Login'),
)
```

#
#### Flutter Async In initState
```dart
  @override
  void initState() {
    super.initState();

    Future.delayed(const Duration(microseconds: 10), () {

      initTimer();
    });
  }

  void initTimer() async{
    String date = await sharePreferencesManager.getString("date");
    if(date.isEmpty){
      await sharePreferencesManager.putString("date", DateTime.now().day.toString());
    }
    timer = Timer.periodic(const Duration(seconds: 1), (Timer t) async {
      DateTime now = DateTime.now();
      // String _time = "${now.hour}:${now.minute}:${now.second}";
      // if(_time=="0:0:1"){
      //   _menuBloc.add(RefreshData());
      // }
      print(t.tick);
      if(date!=DateTime.now().day.toString()){
        await sharePreferencesManager.putString("date", DateTime.now().day.toString());

        Platform.isAndroid ? showAlertDialogLogoutAndroid(context, sharePreferencesManager, "Session Anda expired \n Silahkan login lagi") : showAlertDialogLogoutIos(context, sharePreferencesManager, "Session Anda expired \n Silahkan login lagi");
      }
    });
  }
```

#
#### BaseResponseObject

[Source](https://www.youtube.com/watch?v=wJnMhLLpAIo&ab_channel=PrieyudhaAkaditaS)
```dart
  T? data;

  factory BaseResponseObject.fromJson(Map<String, dynamic> json, Function(Map<String, dynamic>) build) {
    final status = json['status'];
    final title = json['title'];
    final message = json['message'];
    final info = json['info'] != null ? Info.fromJson(json['info']) : null;

    return BaseResponseObject<T>(
      status: status,
      title: title,
      message: message,
      info: info,
      data: build(json['data'])
    );
  }
```
```dart
      return BaseResponseObject<Examples>.fromJson(json.data, (data){
        return Examples.fromJson(data);
      });
```

#
#### BaseResponseList

[Source](https://www.youtube.com/watch?v=wJnMhLLpAIo&ab_channel=PrieyudhaAkaditaS)
```dart
  List<T>? data;

  factory BaseResponseList.fromJson(Map<String, dynamic> json, Function(List<dynamic>) build) {
    final status = json['status'];
    final title = json['title'];
    final message = json['message'];
    final info = json['info'] != null ? Info.fromJson(json['info']) : null;

    return BaseResponseList<T>(
        status: status,
        title: title,
        message: message,
        info: info,
        data: build(json['data'])
    );
  }
```
```dart
return BaseResponseList.fromJson(json.data, (data){
    List<Examples> list = data.map((e) => Examples.fromJson(e)).toList();
    return list;
});
```

#
#### Format
```
String _formatTime(DateTime dateTime) {
    return DateFormat('hh:mm:ss').format(dateTime);
}
```

#
#### Wrap_content
```dart
Wrap(
  children: [
    Text('Do you want to remove item?'),
    Text('Do you want to remove item?'),
    Text('Do you want to remove item?'),
  ],
)
```

#
#### Args
```dart
Navigator.of(context).pop('data');
```
```dart
showDialog(
  context: context,
  builder: (context) => ExamplesDialog(),
).then((value) {
    print(value);
});
```

#
#### Pop
```dart
Navigator.of(context).pop(_editTextController.text);
``

#
#### Navigator
```dart
Navigator.pushNamed(context, NotificationDetailView.TAG, arguments : data);
```
```dart
class AppRouter {
  static Route<dynamic> generateRoute(RouteSettings routeSettings) {
    switch (routeSettings.name) {
      case NotificationDetailView.TAG:
        final args = routeSettings.arguments as NotificationResponse;
        return MaterialPageRoute(builder: (_) => NotificationDetailView(args));
    }
  }
}
```

#
#### WillPopScope

[Source](https://blog.logrocket.com/using-willpopscope-flutter-android-navigation/)

```dart
  @override
  Widget build(BuildContext context) {
    return WillPopScope(
      onWillPop: () async {
        myprint("konfrmasi dulu sebepum acc");
        BlocProvider.of<NotificationBloc>(context).add(NotificationInitEvent());
        return false;
      },
      child: Scaffold(
        appBar: AppBar(
          title: const Text("Notifikasi"),
        ),
        body: Container(),
      ),
    );
  }
```
```dart
Navigator.pushNamed(
  context,
  NotificationDetailView.TAG,
  arguments: c.data.data![index],
).then((value) => setState(() {
  BlocProvider.of<NotificationBloc>(context).add(NotificationInitEvent());
}));
```
```dart
Navigator.pushNamed(
  context,
  NotificationDetailView.TAG,
  arguments: c.data.data![index],
).then((value) {
  BlocProvider.of<NotificationBloc>(context).add(NotificationInitEvent());
});
```

#
#### Base Style Theme
```dart
Theme.of(context).copyWith(splashColor: Colors.yellow);
```
```dart
Theme.of(context).primarySwatch;
```

---

```
Copyright 2022 M. Fadli Zein
```