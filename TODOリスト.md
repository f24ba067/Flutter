import 'package:flutter/material.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'TODOリスト',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        canvasColor: const Color(0xFFfafafa),
      ),
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatefulWidget {
  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  List<bool> _checked = List.generate(10, (_) => false); // チェックボックスの状態
  List<TextEditingController> _controllers = List.generate(
    10,
    (_) => TextEditingController(),
  );

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('TODOリスト')),
      body: ListView.builder(
        itemCount: _checked.length,
        itemBuilder: (context, index) {
          return Row(
            children: [
              Checkbox(
                value: _checked[index],
                onChanged: (bool? value) {
                  setState(() {
                    _checked[index] = value ?? false;
                  });
                },
              ),
              Expanded(
                child: TextField(
                  controller: _controllers[index],
                  decoration: InputDecoration(hintText: 'タスク内容を入力'),
                  style: TextStyle(fontSize: 14),
                ),
              ),
            ],
          );
        },
      ),
    );
  }
}
