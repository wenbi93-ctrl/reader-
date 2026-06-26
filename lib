main.dart
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;
import 'dart:convert';
import 'package:shared_preferences/shared_preferences.dart';
import 'package:flutter_spinkit/flutter_spinkit.dart';

void main() => runApp(TomatoReaderApp());

class TomatoReaderApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: '公版番茄',
      theme: ThemeData.light().copyWith(
        primaryColor: Colors.redAccent,
        scaffoldBackgroundColor: Colors.grey[50],
      ),
      darkTheme: ThemeData.dark().copyWith(
        primaryColor: Colors.black,
        scaffoldBackgroundColor: Colors.black,
      ),
      themeMode: ThemeMode.system, // 跟随系统，但阅读器内部独立控制
      home: MainHomePage(),
      debugShowCheckedModeBanner: false,
    );
  }
}

// ==================== 主界面（底部导航 番茄风格） ====================
class MainHomePage extends StatefulWidget {
  @override
  _MainHomePageState createState() => _MainHomePageState();
}

class _MainHomePageState extends State<MainHomePage> {
  int _currentIndex = 0;
  final List<Widget> _pages = [];

  @override
  void initState() {
    super.initState();
    // 延迟加载保证 context 可用
    WidgetsBinding.instance.addPostFrameCallback((_) {
      setState(() {
        _pages.clear();
        _pages.add(BookShelfPage());
        _pages.add(BookStorePage());
        _pages.add(ProfilePage());
      });
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: _pages.isEmpty
          ? Center(child: SpinKitFadingCircle(color: Colors.redAccent))
          : _pages[_currentIndex],
      bottomNavigationBar: BottomNavigationBar(
        currentIndex: _currentIndex,
        type: BottomNavigationBarType.fixed,
        selectedItemColor: Colors.redAccent,
        unselectedItemColor: Colors.grey,
        onTap: (index) => setState(() => _currentIndex = index),
        items: const [
          BottomNavigationBarItem(icon: Icon(Icons.book), label: '书架'),
          BottomNavigationBarItem(icon: Icon(Icons.explore), label: '书城'),
          BottomNavigationBarItem(icon: Icon(Icons.person), label: '我的'),
        ],
      ),
    );
  }
}

// ==================== 1. 书架页面（支持收藏） ====================
class BookShelfPage extends StatefulWidget {
  @override
  _BookShelfPageState createState() => _BookShelfPageState();
}

class _BookShelfPageState extends State<BookShelfPage> {
  List<Map<String, dynamic>> favoriteBooks = [];
  bool isLoading = true;

  @override
  void initState() {
    super.initState();
    _loadFavorites();
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    // 每次切换到书架时刷新
    _loadFavorites();
  }

  Future<void> _loadFavorites() async {
    final prefs = await SharedPreferences.getInstance();
    final List<String>? ids = prefs.getStringList('favorites');
    if (ids == null || ids.isEmpty) {
      setState(() { favoriteBooks = []; isLoading = false; });
      return;
    }
    setState(() => isLoading = true);
    // 从本地缓存读取（简化：直接存储书籍简略信息，不实时请求API）
    // 实际开发中建议存json字符串
    List<Map<String, dynamic>> temp = [];
    for (String id in ids) {
      // 从prefs读取缓存的书籍信息
      String? bookJson = prefs.getString('book_$id');
      if (bookJson != null) {
        temp.add(json.decode(bookJson));
      }
    }
    setState(() {
      favoriteBooks = temp;
      isLoading = false;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('📚 我的书架', style: TextStyle(fontWeight: FontWeight.bold)),
        backgroundColor: Colors.white,
        foregroundColor: Colors.black,
        elevation: 0,
      ),
      body: isLoading
          ? Center(child: SpinKitFadingCircle(color: Colors.redAccent))
          : favoriteBooks.isEmpty
              ? Center(
                  child: Column(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Icon(Icons.books, size: 80, color: Colors.grey[300]),
                      SizedBox(height: 10),
                      Text('书架空空如也', style: TextStyle(color: Colors.grey)),
                      Text('去书城找点好书吧', style: TextStyle(color: Colors.grey[400])),
                    ],
                  ),
                )
              : ListView.builder(
                  itemCount: favoriteBooks.length,
                  itemBuilder: (ctx, index) {
                    final book = favoriteBooks[index];
                    return _buildBookTile(context, book, isFavorite: true);
                  },
                ),
    );
  }

  Widget _buildBookTile(BuildContext context, Map<String, dynamic> book, {bool isFavorite = false}) {
    final title = book['title'] ?? '未知';
    final authors = book['authors'] as List?;
    final author = (authors != null && authors.isNotEmpty) ? authors[0]['name'] ?? '佚名' : '佚名';
    final formats = book['formats'] ?? {};
    final hasText = formats.containsKey('text/plain');

    return Card(
      margin: EdgeInsets.symmetric(horizontal: 12, vertical: 4),
      child: ListTile(
        leading: CircleAvatar(
          backgroundColor: Colors.redAccent.withOpacity(0.1),
          child: Text(title.substring(0, 1), style: TextStyle(color: Colors.redAccent)),
        ),
        title: Text(title, maxLines: 1, overflow: TextOverflow.ellipsis),
        subtitle: Text('作者：$author'),
        trailing: Row(
          mainAxisSize: MainAxisSize.min,
          children: [
            if (isFavorite)
              IconButton(
                icon: Icon(Icons.favorite, color: Colors.redAccent),
                onPressed: () async {
                  // 取消收藏
                  final prefs = await SharedPreferences.getInstance();
                  List<String>? ids = prefs.getStringList('favorites') ?? [];
                  ids.remove(book['id'].toString());
                  await prefs.setStringList('favorites', ids);
                  await prefs.remove('book_${book['id']}');
                  _loadFavorites();
                },
              ),
            if (hasText)
              Icon(Icons.chevron_right, color: Colors.grey)
            else
              Icon(Icons.block, color: Colors.grey),
          ],
        ),
        onTap: hasText
            ? () => Navigator.push(
                context,
                MaterialPageRoute(
                  builder: (_) => ReaderPage(
                    title: title,
                    contentUrl: formats['text/plain'],
                    bookId: book['id'].toString(),
                  ),
                ),
              ).then((_) => _loadFavorites()) // 返回时刷新进度
            : null,
      ),
    );
  }
}

// ==================== 2. 书城页面（搜索 + 推荐） ====================
class BookStorePage extends StatefulWidget {
  @override
  _BookStorePageState createState() => _BookStorePageState();
}

class _BookStorePageState extends State<BookStorePage> {
  List<dynamic> books = [];
  bool isLoading = false;
  String currentQuery = '科幻'; // 默认推荐

  final TextEditingController _searchController = TextEditingController();

  @override
  void initState() {
    super.initState();
    _searchBooks('科幻');
  }

  Future<void> _searchBooks(String query) async {
    if (query.trim().isEmpty) return;
    setState(() {
      isLoading = true;
      currentQuery = query;
    });
    try {
      final response = await http.get(
        Uri.parse('https://gutendex.com/books?search=$query'),
      );
      if (response.statusCode == 200) {
        final data = json.decode(response.body);
        setState(() {
          books = data['results'] ?? [];
          isLoading = false;
        });
      }
    } catch (e) {
      setState(() => isLoading = false);
      print(e);
    }
  }

  Future<void> _toggleFavorite(Map<String, dynamic> book) async {
    final prefs = await SharedPreferences.getInstance();
    List<String>? ids = prefs.getStringList('favorites') ?? [];
    String id = book['id'].toString();
    if (ids.contains(id)) {
      ids.remove(id);
      await prefs.remove('book_$id');
    } else {
      ids.add(id);
      // 缓存书籍信息
      await prefs.setString('book_$id', json.encode(book));
    }
    await prefs.setStringList('favorites', ids);
    setState(() {}); // 刷新图标
  }

  bool _isFavorite(String id) {
    // 简单判断，实际可以优化性能
    return false; // 后续在build里动态判断比较麻烦，简化在点击时刷新
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('📖 发现好书', style: TextStyle(fontWeight: FontWeight.bold)),
        backgroundColor: Colors.white,
        foregroundColor: Colors.black,
        elevation: 0,
        bottom: PreferredSize(
          preferredSize: Size.fromHeight(60),
          child: Padding(
            padding: const EdgeInsets.all(8.0),
            child: Row(
              children: [
                Expanded(
                  child: TextField(
                    controller: _searchController,
                    decoration: InputDecoration(
                      hintText: '搜索公版书（如：机器人、简爱）',
                      prefixIcon: Icon(Icons.search, color: Colors.grey),
                      border: OutlineInputBorder(
                        borderRadius: BorderRadius.circular(30),
                        borderSide: BorderSide.none,
                      ),
                      filled: true,
                      fillColor: Colors.grey[200],
                    ),
                    onSubmitted: _searchBooks,
                  ),
                ),
                SizedBox(width: 8),
                IconButton(
                  icon: Icon(Icons.refresh),
                  onPressed: () => _searchBooks(_searchController.text.isNotEmpty ? _searchController.text : '科幻'),
                ),
              ],
            ),
          ),
        ),
      ),
      body: isLoading
          ? Center(child: SpinKitFadingCircle(color: Colors.redAccent))
          : books.isEmpty
              ? Center(child: Text('没找到相关书籍，换个关键词试试'))
              : ListView.builder(
                  itemCount: books.length,
                  itemBuilder: (ctx, index) {
                    final book = books[index];
                    final title = book['title'] ?? '未知';
                    final authors = book['authors'] as List?;
                    final author = (authors != null && authors.isNotEmpty) ? authors[0]['name'] ?? '佚名' : '佚名';
                    final formats = book['formats'] ?? {};
                    final hasText = formats.containsKey('text/plain');
                    final subjects = book['subjects'] as List?;
                    final subjectStr = (subjects != null && subjects.isNotEmpty) ? subjects[0] : '公版经典';

                    return Card(
                      margin: EdgeInsets.symmetric(horizontal: 12, vertical: 4),
                      child: ListTile(
                        leading: CircleAvatar(
                          backgroundColor: Colors.redAccent.withOpacity(0.1),
                          child: Text(title.substring(0, 1), style: TextStyle(color: Colors.redAccent)),
                        ),
                        title: Text(title, maxLines: 1, overflow: TextOverflow.ellipsis),
                        subtitle: Text('$author · $subjectStr'),
                        trailing: Row(
                          mainAxisSize: MainAxisSize.min,
                          children: [
                            // 仿番茄 收藏按钮
                            FutureBuilder<List<String>?>(
                              future: SharedPreferences.getInstance().then((prefs) => prefs.getStringList('favorites')),
                              builder: (ctx, snapshot) {
                                bool isFav = false;
                                if (snapshot.hasData && snapshot.data != null) {
                                  isFav = snapshot.data!.contains(book['id'].toString());
                                }
                                return IconButton(
                                  icon: Icon(
                                    isFav ? Icons.favorite : Icons.favorite_border,
                                    color: isFav ? Colors.redAccent : Colors.grey,
                                  ),
                                  onPressed: () => _toggleFavorite(book).then((_) => setState(() {})),
                                );
                              },
                            ),
                            if (hasText)
                              Icon(Icons.chevron_right, color: Colors.grey)
                            else
                              Icon(Icons.block, color: Colors.grey),
                          ],
                        ),
                        onTap: hasText
                            ? () => Navigator.push(
                                context,
                                MaterialPageRoute(
                                  builder: (_) => ReaderPage(
                                    title: title,
                                    contentUrl: formats['text/plain'],
                                    bookId: book['id'].toString(),
                                  ),
                                ),
                              )
                            : null,
                      ),
                    );
                  },
                ),
    );
  }
}

// ==================== 3. 阅读页面（番茄风：夜间模式+进度保存） ====================
class ReaderPage extends StatefulWidget {
  final String title;
  final String contentUrl;
  final String bookId;

  const ReaderPage({required this.title, required this.contentUrl, required this.bookId});

  @override
  _ReaderPageState createState() => _ReaderPageState();
}

class _ReaderPageState extends State<ReaderPage> {
  String content = '';
  bool isLoading = true;
  double fontSize = 20.0;
  bool isDarkMode = false;
  double scrollProgress = 0.0;
  final ScrollController _scrollController = ScrollController();

  @override
  void initState() {
    super.initState();
    _loadContent();
    _loadProgress();
    _scrollController.addListener(_updateProgress);
  }

  @override
  void dispose() {
    _scrollController.dispose();
    super.dispose();
  }

  Future<void> _loadProgress() async {
    final prefs = await SharedPreferences.getInstance();
    double? saved = prefs.getDouble('progress_${widget.bookId}');
    if (saved != null) {
      setState(() => scrollProgress = saved);
    }
  }

  void _updateProgress() {
    if (_scrollController.hasClients) {
      double maxScroll = _scrollController.position.maxScrollExtent;
      double currentScroll = _scrollController.position.pixels;
      if (maxScroll > 0) {
        double progress = currentScroll / maxScroll;
        if ((progress - scrollProgress).abs() > 0.01) {
          setState(() => scrollProgress = progress);
          _saveProgress(progress);
        }
      }
    }
  }

  Future<void> _saveProgress(double progress) async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setDouble('progress_${widget.bookId}', progress);
  }

  Future<void> _loadContent() async {
    setState(() => isLoading = true);
    try {
      final response = await http.get(Uri.parse(widget.contentUrl));
      if (response.statusCode == 200) {
        String raw = response.body;
        if (raw.length > 5000) raw = raw.substring(0, 5000) + '\n\n... (已截断展示开头)';
        setState(() { content = raw; isLoading = false; });
        // 加载完成后跳转到上次进度
        WidgetsBinding.instance.addPostFrameCallback((_) {
          if (_scrollController.hasClients && scrollProgress > 0) {
            _scrollController.jumpTo(_scrollController.position.maxScrollExtent * scrollProgress);
          }
        });
      }
    } catch (e) {
      setState(() { content = '加载失败：$e'; isLoading = false; });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: isDarkMode ? Colors.black : Color(0xFFF5F0E6),
      appBar: AppBar(
        title: Text(widget.title, style: TextStyle(fontSize: 16)),
        backgroundColor: isDarkMode ? Colors.grey[900] : Colors.white,
        foregroundColor: isDarkMode ? Colors.white : Colors.black,
        elevation: 0,
        actions: [
          // 字号调整
          PopupMenuButton<double>(
            icon: Icon(Icons.text_fields),
            onSelected: (val) => setState(() => fontSize = val),
            itemBuilder: (ctx) => [14, 18, 22, 26, 30, 34]
                .map((size) => PopupMenuItem(value: size.toDouble(), child: Text('$size px')))
                .toList(),
          ),
          // 夜间模式切换
          IconButton(
            icon: Icon(isDarkMode ? Icons.light_mode : Icons.dark_mode),
            onPressed: () => setState(() => isDarkMode = !isDarkMode),
          ),
        ],
        bottom: PreferredSize(
          preferredSize: Size.fromHeight(2),
          child: LinearProgressIndicator(
            value: scrollProgress,
            backgroundColor: Colors.grey[300],
            color: Colors.redAccent,
          ),
        ),
      ),
      body: isLoading
          ? Center(child: SpinKitFadingCircle(color: Colors.redAccent))
          : Padding(
              padding: EdgeInsets.symmetric(horizontal: 16, vertical: 20),
              child: SingleChildScrollView(
                controller: _scrollController,
                child: Text(
                  content,
                  style: TextStyle(
                    fontSize: fontSize,
                    height: 1.8,
                    color: isDarkMode ? Colors.grey[300] : Colors.black87,
                  ),
                ),
              ),
            ),
      floatingActionButton: Container(
        margin: EdgeInsets.only(bottom: 30),
        child: FloatingActionButton.small(
          backgroundColor: isDarkMode ? Colors.grey[800] : Colors.white,
          child: Icon(Icons.close, color: isDarkMode ? Colors.white : Colors.black),
          onPressed: () => Navigator.pop(context),
        ),
      ),
      floatingActionButtonLocation: FloatingActionButtonLocation.centerFloat,
    );
  }
}

// ==================== 4. 我的页面（占位） ====================
class ProfilePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('我的', style: TextStyle(fontWeight: FontWeight.bold)),
        backgroundColor: Colors.white,
        foregroundColor: Colors.black,
        elevation: 0,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Icon(Icons.account_circle, size: 100, color: Colors.grey[300]),
            SizedBox(height: 20),
            Text('登录后体验更多功能', style: TextStyle(color: Colors.grey)),
            SizedBox(height: 10),
            Text('书评、书单、云同步开发中...', style: TextStyle(color: Colors.grey[400], fontSize: 12)),
          ],
        ),
      ),
    );
  }
}