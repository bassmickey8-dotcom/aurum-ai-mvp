# aurum-ai-mvp
MVP for AI-powered gold-finding app
import 'package:flutter/material.dart';
import 'package:sensors_plus/sensors_plus.dart';
import 'dart:math';
import 'package:image_picker/image_picker.dart';
import 'dart:io';

void main() => runApp(const AurumApp());

class AurumApp extends StatelessWidget {
  const AurumApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Aurum AI',
      theme: ThemeData.dark().copyWith(
        primaryColor: Colors.amber,
        scaffoldBackgroundColor: Colors.black,
      ),
      home: const GoldScanner(),
    );
  }
}

class GoldScanner extends StatefulWidget {
  const GoldScanner({super.key});

  @override
  State<GoldScanner> createState() => _GoldScannerState();
}

class _GoldScannerState extends State<GoldScanner> {
  double _magStrength = 0.0;
  String _status = "Point at dirt...";
  File? _pickedImage;

  @override
  void initState() {
    super.initState();
    magnetometerEvents.listen((MagnetometerEvent event) {
      double strength = sqrt(event.x * event.x + event.y * event.y + event.z * event.z);
      setState(() {
        _magStrength = strength;
        if (strength > 55) {
          _status = "Spike! Possible metal nearby";
        } else {
          _status = "Quiet... keep scanning";
        }
      });
    });
  }

  Future<void> _pickImage() async {
    final picker = ImagePicker();
    final XFile? image = await picker.pickImage(source: ImageSource.camera);
    if (image != null) {
      setState(() {
        _pickedImage = File(image.path);
        _status = "Photo taken—AI analysis coming soon";
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Aurum AI', style: TextStyle(color: Colors.amber)),
        backgroundColor: Colors.grey[900],
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text(
              "Magnetic field: ${_magStrength.toStringAsFixed(1)} μT",
              style: const TextStyle(fontSize: 32, color: Colors.amber),
            ),
            const SizedBox(height: 20),
            Text(
              _status,
              style: const TextStyle(fontSize: 18, color: Colors.grey),
              textAlign: TextAlign.center,
            ),
            const SizedBox(height: 40),
            if (_pickedImage != null)
              Image.file(_pickedImage!, height: 200, fit: BoxFit.cover),
            const SizedBox(height: 20),
            ElevatedButton.icon(
              onPressed: _pickImage,
              icon: const Icon(Icons.camera_alt),
              label: const Text('Snap a rock'),
              style: ElevatedButton.styleFrom(
                backgroundColor: Colors.amber,
                foregroundColor: Colors.black,
                padding: const EdgeInsets.symmetric(horizontal: 30, vertical: 15),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
