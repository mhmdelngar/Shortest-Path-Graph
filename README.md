# Shortest-Path-Graph
import 'package:flutter/material.dart';

void main() => runApp(const MaterialApp(home: ShortestPathGraph()));

class Node {
  final String id;
  Offset position;
  Node({required this.id, required this.position});
}

class Edge {
  final String startId;
  final String endId;
  Edge(this.startId, this.endId);
}

class ShortestPathGraph extends StatefulWidget {
  const ShortestPathGraph({super.key});

  @override
  State<ShortestPathGraph> createState() => _ShortestPathGraphState();
}

class _ShortestPathGraphState extends State<ShortestPathGraph> {
  // 1. Initial Data
  final List<Node> nodes = [
    Node(id: 'A', position: const Offset(100, 300)),
    Node(id: 'B', position: const Offset(300, 150)),
    Node(id: 'C', position: const Offset(300, 450)),
    Node(id: 'D', position: const Offset(500, 150)),
    Node(id: 'E', position: const Offset(500, 450)),
    Node(id: 'F', position: const Offset(700, 300)),
  ];

  final List<Edge> edges = [
    Edge('A', 'B'),
    Edge('A', 'C'),
    Edge('B', 'D'),
    Edge('C', 'E'),
    Edge('D', 'F'),
    Edge('E', 'F'),
    Edge('B', 'E'),
    Edge('C', 'D'),
  ];

  List<String> path = [];
  String startNodeId = 'A';
  String endNodeId = 'F';

  @override
  void initState() {
    super.initState();
    _calculatePath();
  }

  // 2. Dijkstra Algorithm
  void _calculatePath() {
    Map<String, double> dists = {for (var n in nodes) n.id: double.infinity};
    Map<String, String?> prevs = {for (var n in nodes) n.id: null};
    List<String> unvisited = nodes.map((n) => n.id).toList();

    dists[startNodeId] = 0;

    while (unvisited.isNotEmpty) {
      unvisited.sort((a, b) => dists[a]!.compareTo(dists[b]!));
      String curr = unvisited.removeAt(0);
      if (curr == endNodeId || dists[curr] == double.infinity) break;

      for (var edge in edges) {
        String? neighbor;
        if (edge.startId == curr)
          neighbor = edge.endId;
        else if (edge.endId == curr)
          neighbor = edge.startId;

        if (neighbor != null && unvisited.contains(neighbor)) {
          double weight =
              (nodes.firstWhere((n) => n.id == curr).position -
                      nodes.firstWhere((n) => n.id == neighbor).position)
                  .distance;
          double alt = dists[curr]! + weight;
          if (alt < dists[neighbor]!) {
            dists[neighbor] = alt;
            prevs[neighbor] = curr;
          }
        }
      }
    }

    List<String> newPath = [];
    String? temp = endNodeId;
    while (temp != null) {
      newPath.insert(0, temp);
      temp = prevs[temp];
    }
    setState(() => path = newPath.first == startNodeId ? newPath : []);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.grey[200],
      appBar: AppBar(title: const Text("Shortest Path Graph (A to F)")),
      body: InteractiveViewer(
        constrained: false, // Allows the canvas to be larger than the screen
        boundaryMargin: const EdgeInsets.all(500),
        minScale: .1,
        maxScale: 5.0,
        child: Stack(
          children: [
            // Painter for lines and weights
            CustomPaint(
              size: const Size(2000, 2000),
              painter: GraphPainter(nodes: nodes, edges: edges, path: path),
            ),
            // Draggable Nodes
            ...nodes.map(
              (node) => Positioned(
                left: node.position.dx - 25,
                top: node.position.dy - 25,
                child: GestureDetector(
                  onPanUpdate: (details) {
                    setState(() {
                      node.position += details.delta;
                      _calculatePath();
                    });
                  },
                  child: CircleAvatar(
                    radius: 25,
                    backgroundColor: path.contains(node.id)
                        ? Colors.orange
                        : Colors.blue,
                    child: Text(
                      node.id,
                      style: const TextStyle(color: Colors.white),
                    ),
                  ),
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class GraphPainter extends CustomPainter {
  final List<Node> nodes;
  final List<Edge> edges;
  final List<String> path;

  GraphPainter({required this.nodes, required this.edges, required this.path});

  @override
  void paint(Canvas canvas, Size size) {
    final linePaint = Paint()
      ..color = Colors.black26
      ..strokeWidth = 2;
    final pathPaint = Paint()
      ..color = Colors.orange
      ..strokeWidth = 4;

    for (var edge in edges) {
      final n1 = nodes.firstWhere((n) => n.id == edge.startId);
      final n2 = nodes.firstWhere((n) => n.id == edge.endId);

      bool isPath = false;
      for (int i = 0; i < path.length - 1; i++) {
        if ((path[i] == n1.id && path[i + 1] == n2.id) ||
            (path[i] == n2.id && path[i + 1] == n1.id)) {
          isPath = true;
          break;
        }
      }

      canvas.drawLine(n1.position, n2.position, isPath ? pathPaint : linePaint);

      // Draw Distance Label
      final distance = (n1.position - n2.position).distance.toStringAsFixed(0);
      final tp = TextPainter(
        text: TextSpan(
          text: distance,
          style: const TextStyle(
            color: Colors.black,
            fontSize: 10,
            backgroundColor: Colors.white70,
          ),
        ),
        textDirection: TextDirection.ltr,
      )..layout();
      tp.paint(
        canvas,
        Offset(
          (n1.position.dx + n2.position.dx) / 2 - 10,
          (n1.position.dy + n2.position.dy) / 2 - 5,
        ),
      );
    }
  }

  @override
  bool shouldRepaint(GraphPainter old) => true;
}
