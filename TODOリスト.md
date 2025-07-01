```
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

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

class TodoItem {
  late String id;
  String text;
  bool isDone;
  late TextEditingController controller;
  late FocusNode focusNode;

  TodoItem({String? id, this.text = '', this.isDone = false}) {
    this.id = id ?? UniqueKey().toString();
    controller = TextEditingController(text: text);
    focusNode = FocusNode();
  }

  void dispose() {
    controller.dispose();
    focusNode.dispose();
  }

  factory TodoItem.fromJson(Map<String, dynamic> json) {
    return TodoItem(id: json['id'], text: json['text'], isDone: json['isDone']);
  }

  Map<String, dynamic> toJson() => {
    'id': id,
    'text': controller.text, 
    'isDone': isDone,
  };
}

class MyHomePage extends StatefulWidget {
  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  List<TodoItem> _todoItems = [];

  @override
  void initState() {
    super.initState();
    _loadTodoItems(); 
  }

  @override
  void dispose() {
    for (var item in _todoItems) {
      item.dispose();
    }
    super.dispose();
  }

  Future<void> _loadTodoItems() async {
    final prefs = await SharedPreferences.getInstance();
    final String? todosString = prefs.getString('todoItems');
    if (todosString != null) {
      try {
        final List<dynamic> todoListJson = json.decode(todosString);
        setState(() {
          _todoItems = todoListJson
              .map(
                (jsonItem) =>
                    TodoItem.fromJson(jsonItem as Map<String, dynamic>),
              )
              .toList();
        });
      } catch (e) {
        print("Error decoding todoItems: $e. Data: $todosString");
        await prefs.remove('todoItems');
        setState(() {
          _todoItems = [];
        });
      }
    }
  }

  Future<void> _saveTodoItems() async {
    final prefs = await SharedPreferences.getInstance();
    List<Map<String, dynamic>> todoListJson = _todoItems
        .map((item) => item.toJson())
        .toList();
    await prefs.setString('todoItems', json.encode(todoListJson));
  }

  void _addTodoItem({
    String initialText = '',
    bool isDone = false,
    bool requestFocus = false,
  }) {
    final newItem = TodoItem(text: initialText, isDone: isDone);
    setState(() {
      _todoItems.add(newItem);
    });
    _saveTodoItems();
    if (requestFocus && mounted) {
      Future.delayed(Duration(milliseconds: 50), () {
        if (mounted && newItem.focusNode.canRequestFocus) {
          FocusScope.of(context).requestFocus(newItem.focusNode);
        }
      });
    }
  }

  // タスクを削除するメソッド
  void _removeTodoItem(int index) {
    if (index < 0 || index >= _todoItems.length) return; // 範囲外なら何もしない
    final removedItem = _todoItems[index];
    removedItem.dispose(); // 削除するアイテムのリソースを破棄
    setState(() {
      _todoItems.removeAt(index);
    });
    _saveTodoItems();
  }

  void _toggleTodoStatus(int index, bool? value) {
    if (index < 0 || index >= _todoItems.length) return; // 範囲外なら何もしない
    setState(() {
      _todoItems[index].isDone = value ?? false;
    });
    _saveTodoItems();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('TODOリスト')),
      body: _todoItems.isEmpty
          ? Center(child: Text('タスクがありません。追加しましょう！'))
          : ListView.builder(
              padding: EdgeInsets.only(bottom: 80), 
              itemCount: _todoItems.length,
              itemBuilder: (context, index) {
                final todo = _todoItems[index];
                return Dismissible(
                  key: Key(todo.id), 
                  direction: DismissDirection.endToStart, 
                  onDismissed: (direction) {
                    String taskText = todo.controller.text.isNotEmpty
                        ? todo.controller.text
                        : "タスク";
                    _removeTodoItem(index);
                    ScaffoldMessenger.of(context).showSnackBar(
                      SnackBar(content: Text('「$taskText」を削除しました')),
                    );
                  },
                  background: Container(
                    color: Colors.redAccent,
                    alignment: Alignment.centerRight,
                    padding: EdgeInsets.symmetric(horizontal: 20.0),
                    child: Icon(
                      Icons.delete_sweep_outlined,
                      color: Colors.white,
                    ),
                  ),
                  child: Row(
                    children: [
                      Checkbox(
                        value: todo.isDone,
                        onChanged: (bool? value) =>
                            _toggleTodoStatus(index, value),
                      ),
                      Expanded(
                        child: TextField(
                          controller: todo.controller,
                          focusNode: todo.focusNode,
                          decoration: InputDecoration(
                            hintText: 'タスク内容を入力',
                            border: InputBorder.none, 
                          ),
                          style: TextStyle(
                            fontSize: 16,
                            decoration: todo.isDone
                                ? TextDecoration.lineThrough
                                : TextDecoration.none,
                            color: todo.isDone ? Colors.grey : Colors.black87,
                          ),
                          onSubmitted: (value) {
                            _saveTodoItems(); 
                            _addTodoItem(
                              requestFocus: true,
                            ); 
                          },
                          onChanged: (value) {
                          },
                        ),
                      ),
                    ],
                  ),
                );
              },
            ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _addTodoItem(requestFocus: true), 
        tooltip: 'タスクを追加',
        child: Icon(Icons.add),
      ),
    );
  }
}

```
