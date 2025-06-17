```
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

// Todoアイテムを表すクラス
class TodoItem {
  String id;
  String text;
  bool isDone;
  TextEditingController controller;

  TodoItem({String? id, this.text = '', this.isDone = false})
    : this.id = id ?? UniqueKey().toString(), // IDがなければユニークなIDを生成
      controller = TextEditingController(text: text);

  // TextEditingControllerを破棄するメソッド
  void dispose() {
    controller.dispose();
  }
}

class MyHomePage extends StatefulWidget {
  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  List<TodoItem> _todoItems = []; // TodoItemのリスト

  @override
  void initState() {
    super.initState();
    // 初期データをいくつか追加する場合
    // _addTodoItem('最初のタスク');
    // _addTodoItem('次のタスク', isDone: true);
  }

  @override
  void dispose() {
    // 全てのTodoItemのコントローラーを破棄
    for (var item in _todoItems) {
      item.dispose();
    }
    super.dispose();
  }

  // 新しいタスクを追加するメソッド
  void _addTodoItem([String initialText = '', bool isDone = false]) {
    setState(() {
      _todoItems.add(TodoItem(text: initialText, isDone: isDone));
    });
  }

  // タスクを削除するメソッド
  void _removeTodoItem(int index) {
    _todoItems[index].dispose(); // 削除するアイテムのコントローラーを破棄
    setState(() {
      _todoItems.removeAt(index);
    });
  }

  // タスクの完了状態を切り替えるメソッド
  void _toggleTodoStatus(int index, bool? value) {
    setState(() {
      _todoItems[index].isDone = value ?? false;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('TODOリスト')),
      body: _todoItems.isEmpty
          ? Center(child: Text('タスクがありません。追加しましょう！'))
          : ListView.builder(
              itemCount: _todoItems.length,
              itemBuilder: (context, index) {
                final todo = _todoItems[index];
                return Row(
                  children: [
                    Checkbox(
                      value: todo.isDone,
                      onChanged: (bool? value) =>
                          _toggleTodoStatus(index, value),
                    ),
                    Expanded(
                      child: TextField(
                        controller: todo.controller,
                        decoration: InputDecoration(
                          hintText: 'タスク内容を入力',
                          border: InputBorder.none, // 枠線を消してスッキリさせる
                        ),
                        style: TextStyle(
                          fontSize: 16,
                          decoration: todo.isDone
                              ? TextDecoration.lineThrough
                              : TextDecoration.none,
                          color: todo.isDone ? Colors.grey : Colors.black87,
                        ),
                      ),
                    ),
                    IconButton(
                      icon: Icon(Icons.delete_outline, color: Colors.redAccent),
                      onPressed: () => _removeTodoItem(index),
                    ),
                  ],
                );
              },
            ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _addTodoItem(), // 新しい空のタスクを追加
        tooltip: 'タスクを追加',
        child: Icon(Icons.add),
      ),
    );
  }
}

```
