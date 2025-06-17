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

// Todoアイテムを表すクラス
class TodoItem {
  late String id;
  String text;
  bool isDone;
  late TextEditingController controller;
  late FocusNode focusNode;

  TodoItem({String? id, this.text = '', this.isDone = false}) {
    this.id = id ?? UniqueKey().toString(); // IDがなければユニークなIDを生成
    controller = TextEditingController(text: text);
    focusNode = FocusNode();
  }

  // TextEditingControllerとFocusNodeを破棄するメソッド
  void dispose() {
    controller.dispose();
    focusNode.dispose();
  }

  // JSONからTodoItemを生成するファクトリコンストラクタ
  factory TodoItem.fromJson(Map<String, dynamic> json) {
    return TodoItem(id: json['id'], text: json['text'], isDone: json['isDone']);
  }

  // TodoItemをJSONに変換するメソッド
  Map<String, dynamic> toJson() => {
    'id': id,
    'text': controller.text, // 保存時はcontrollerの最新のテキストを使用
    'isDone': isDone,
  };
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
    _loadTodoItems(); // アプリ起動時にデータを読み込む
  }

  @override
  void dispose() {
    // 全てのTodoItemのコントローラーを破棄
    for (var item in _todoItems) {
      item.dispose();
    }
    super.dispose();
  }

  // SharedPreferencesからデータを読み込む
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
        // エラー発生時は、破損した可能性のあるデータをクリアし、リストを空にする
        await prefs.remove('todoItems');
        setState(() {
          _todoItems = [];
        });
      }
    }
  }

  // SharedPreferencesにデータを保存する
  Future<void> _saveTodoItems() async {
    final prefs = await SharedPreferences.getInstance();
    List<Map<String, dynamic>> todoListJson = _todoItems
        .map((item) => item.toJson())
        .toList();
    await prefs.setString('todoItems', json.encode(todoListJson));
  }

  // 新しいタスクを追加するメソッド
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
      // UIのビルド後にフォーカスを要求するため、少し遅延させる
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

  // タスクの完了状態を切り替えるメソッド
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
              padding: EdgeInsets.only(bottom: 80), // FABと重ならないようにパディング
              itemCount: _todoItems.length,
              itemBuilder: (context, index) {
                final todo = _todoItems[index];
                return Dismissible(
                  key: Key(todo.id), // 各DismissibleウィジェットにユニークなKeyを設定
                  direction: DismissDirection.endToStart, // 右から左へのスワイプのみ許可
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
                            border: InputBorder.none, // 枠線を消してスッキリさせる
                          ),
                          style: TextStyle(
                            fontSize: 16,
                            decoration: todo.isDone
                                ? TextDecoration.lineThrough
                                : TextDecoration.none,
                            color: todo.isDone ? Colors.grey : Colors.black87,
                          ),
                          onSubmitted: (value) {
                            _saveTodoItems(); // 現在のタスクの変更を保存
                            _addTodoItem(
                              requestFocus: true,
                            ); // 新しいタスクを追加してフォーカス
                          },
                          onChanged: (value) {
                            // テキストが変更されるたびに保存することも可能ですが、
                            // パフォーマンスへの影響を考慮し、onSubmittedや他のアクション時のみ保存します。
                            // 必要であれば、ここで _saveTodoItems(); を呼び出すこともできます。
                            // _saveTodoItems(); // リアルタイム保存 (頻繁な書き込みに注意)
                          },
                        ),
                      ),
                    ],
                  ),
                );
              },
            ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _addTodoItem(requestFocus: true), // 新しいタスクを追加しフォーカス
        tooltip: 'タスクを追加',
        child: Icon(Icons.add),
      ),
    );
  }
}

```
