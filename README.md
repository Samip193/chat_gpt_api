# chat_gpt_api
import 'dart:convert';
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;

// Model Class
class Post {
  final int userId;
  final int id;
  final String title;
  final String body;

  Post({
    required this.userId,
    required this.id,
    required this.title,
    required this.body,
  });

  // Factory constructor to create a Post from JSON
  factory Post.fromJson(Map<String, dynamic> json) {
    return Post(
      userId: json['userId'],
      id: json['id'],
      title: json['title'],
      body: json['body'],
    );
  }

  // Method to convert Post object to JSON
  Map<String, dynamic> toJson() {
    return {
      'userId': userId,
      'id': id,
      'title': title,
      'body': body,
    };
  }
}

Future<List<Post>> fetchPosts() async {
  final url = Uri.parse('https://jsonplaceholder.typicode.com/posts');

  try {
    final response = await http.get(url);

    if (response.statusCode == 200) {
      final List<dynamic> data = jsonDecode(response.body);
      return data.map((json) => Post.fromJson(json)).toList();
    } else {
      throw Exception('Failed to fetch data. Status code: ${response.statusCode}');
    }
  } catch (e) {
    throw Exception('Error occurred: $e');
  }
}

Future<void> createPost(Post post) async {
  final url = Uri.parse('https://jsonplaceholder.typicode.com/posts');

  try {
    final response = await http.post(
      url,
      headers: {'Content-Type': 'application/json'},
      body: jsonEncode(post.toJson()),
    );

    if (response.statusCode == 201) {
      final createdPost = Post.fromJson(jsonDecode(response.body));
      print('Post created successfully: ${createdPost.title}');
    } else {
      throw Exception('Failed to create post. Status code: ${response.statusCode}');
    }
  } catch (e) {
    throw Exception('Error occurred while creating post: $e');
  }
}

Future<void> updatePost(Post post) async {
  final url = Uri.parse('https://jsonplaceholder.typicode.com/posts/${post.id}');

  try {
    final response = await http.put(
      url,
      headers: {'Content-Type': 'application/json'},
      body: jsonEncode(post.toJson()),
    );

    if (response.statusCode == 200) {
      final updatedPost = Post.fromJson(jsonDecode(response.body));
      print('Post updated successfully: ${updatedPost.title}');
    } else {
      throw Exception('Failed to update post. Status code: ${response.statusCode}');
    }
  } catch (e) {
    throw Exception('Error occurred while updating post: $e');
  }
}

Future<void> deletePost(int postId) async {
  final url = Uri.parse('https://jsonplaceholder.typicode.com/posts/$postId');

  try {
    final response = await http.delete(url);

    if (response.statusCode == 200 || response.statusCode == 204) {
      print('Post deleted successfully');
    } else {
      throw Exception('Failed to delete post. Status code: ${response.statusCode}');
    }
  } catch (e) {
    throw Exception('Error occurred while deleting post: $e');
  }
}

class PostListScreen extends StatelessWidget {
  const PostListScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Post List'),
        actions: [
          IconButton(
            icon: const Icon(Icons.add),
            onPressed: () {
              final newPost = Post(
                userId: 1,
                id: 0,
                title: 'New Post',
                body: 'This is the body of the new post.',
              );
              createPost(newPost);
            },
          ),
        ],
      ),
      body: FutureBuilder<List<Post>>(
        future: fetchPosts(),
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          } else if (snapshot.hasError) {
            return Center(child: Text('Error: ${snapshot.error}'));
          } else if (!snapshot.hasData || snapshot.data!.isEmpty) {
            return const Center(child: Text('No posts found.'));
          } else {
            final posts = snapshot.data!;

            return ListView.builder(
              itemCount: posts.length,
              itemBuilder: (context, index) {
                final post = posts[index];
                return Card(
                  margin: const EdgeInsets.all(8.0),
                  child: ListTile(
                    title: Text(post.title),
                    subtitle: Text(post.body),
                    trailing: Row(
                      mainAxisSize: MainAxisSize.min,
                      children: [
                        IconButton(
                          icon: const Icon(Icons.edit, color: Colors.blue),
                          onPressed: () {
                            final updatedPost = Post(
                              userId: post.userId,
                              id: post.id,
                              title: '${post.title} (Updated)',
                              body: '${post.body} (Updated)',
                            );
                            updatePost(updatedPost);
                          },
                        ),
                        IconButton(
                          icon: const Icon(Icons.delete, color: Colors.red),
                          onPressed: () {
                            deletePost(post.id);
                          },
                        ),
                      ],
                    ),
                  ),
                );
              },
            );
          }
        },
      ),
    );
  }
}

void main() {
  runApp(const MaterialApp(
    home: PostListScreen(),
  ));
}
