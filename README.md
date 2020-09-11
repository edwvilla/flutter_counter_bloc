# BloC main topics

bloc is used to controll streams, events comes in and streams comes out

StreamControllers has 2 main components
- Sink (inputs)
- Stream (outputs)


Steps to build this particular app

* **First step**

Identify the events

for this case a simple counter needs only two events Increment and Decrement

both are events so for better design create an abstract class and inherit it later

```dart
abstract class CounterEvent {}

class IncrementEvent extends CounterEvent {}

class DecrementEvent extends CounterEvent {}
```

* **Second step**

Define attributes

the only attribute used is the counter, it has to be declared as private using underscore `_counter`

then add the StreamController to manage inputs (sink) and outputs (stream) for that attribute

```dart
class CounterBloc {
  int _counter = 0;

  final _counterStateController = StreamController<int>();

  StreamSink<int> get _inCounter => _counterStateController.sink;
  Stream<int> get counter => _counterStateController.stream;
}
```
note: the stream (output) is declared as a public getter cause that is the method used to extract the value

* **Third step**

Define the event to be triggered by the ui

the sink (input) part needs to be declared as a public getter cause the ui will trigger it sending the corresponding event (increment or decrement)

```dart
clas CounterBloc{
...
final _counterEventController = StreamController<CounterEvent>();
  StreamSink<CounterEvent> get counterEventSink => _counterEventController.sink;
  //TODO ADD the stream
}
```
and the stream (output) part needs to be declared in a way it can be listened, so in the constructor to listen what the ui is sending as event
using a conditional to decide if the counter increments or decrements

```dart
clas CounterBloc{
...
 //TODO ADD the stream
  CounterBloc() {
    _counterEventController.stream.listen(_mapEventToState);
  }
  
  void _mapEventToState(CounterEvent event) {
    if (event is IncrementEvent)
      _counter++;
    else
      _counter--;

    _inCounter.add(_counter);
  }
}

```
note: a disposal method needs to be added to prevent memory leaks issues

```dart
void dispose() {
    _counterStateController.close();
    _counterEventController.close();
  }
```

* **Fifth step**

Connecting the ui

```dart

class _MyHomePageState extends State<MyHomePage> {
  final CounterBloc _bloc = CounterBloc(); // create an instance of bloc
 ...
    return StreamBuilder<int>(  // use a stream builder
        stream: _bloc.counter,  // listen to the counter stream getter
        initialData: 0,         // initialize the value
          ...
                  Text('${snapshot.data}', // the value is inside the snapshot.data
            ...
                FloatingActionButton(
                  onPressed: () => _bloc.counterEventSink.add(IncrementEvent()), // send the triggers using the corresponding event instance
                  ...
                FloatingActionButton(
                  onPressed: () => _bloc.counterEventSink.add(DecrementEvent()), // send the triggers using the corresponding event instance
                  ...
  @override
  void dispose() {
    super.dispose();
    _bloc.dispose();
  }
}
```
