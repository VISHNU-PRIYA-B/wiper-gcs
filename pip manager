import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:media_kit_video/media_kit_video.dart';

import '../../modules/wiper-gcs/widgets/camera_view.dart';
import '../../ui/app_theme.dart';

class PipManager {
  static const MethodChannel _channel = MethodChannel('vivid_mapping_sandbox/pip');

  static OverlayEntry? _overlayEntry;

  /// =========================================================
  /// ANDROID NATIVE PiP
  /// =========================================================
  static Future<void> enterNativePip(BuildContext context) async {
    final url = GlobalStreamCache.activeStreamUrl;

    if (url == null) {
      _showSnackBar(context, 'Start a video stream first.', AppTheme.red);
      return;
    }

    try {
      await _channel.invokeMethod('enterPipMode');
    } catch (e) {
      debugPrint('Native PiP failed: $e');

      if (context.mounted) {
        _showSnackBar(
          context,
          'PiP not supported on this device.',
          AppTheme.amber,
        );
      }
    }
  }

  /// =========================================================
  /// IN-APP FLOATING PiP
  /// =========================================================
  static void showInAppPip(BuildContext context, String url) {
    if (_overlayEntry != null) return;

    _overlayEntry = OverlayEntry(builder: (_) => _FloatingPipWidget(url: url));

    Overlay.of(context).insert(_overlayEntry!);
  }

  static void hideInAppPip() {
    _overlayEntry?.remove();
    _overlayEntry = null;
  }

  /// =========================================================
  /// TOGGLE PiP
  /// =========================================================
static Future<void> togglePip(BuildContext context) async{
  if (_overlayEntry != null) {
    hideInAppPip();
    return;
  }

  final urls = GlobalStreamCache.players.keys.toList();
  // final urls = GlobalStreamCache.players.keys.where((url) => allowedUrls.contains(url)).toList();

  if (urls.isEmpty) {
    _showSnackBar(
      context,
      'Start a video stream first.',
      AppTheme.red,
    );
    return;
  }
  debugPrint("Player count: ${GlobalStreamCache.players.length}");
  debugPrint("Players: ${GlobalStreamCache.players.keys.toList()}");

  if (urls.length == 1) {
    // showInAppPip(context, urls.first);
    // GlobalStreamCache.activeStreamUrl = urls.first; // Set the active stream URL
    showInAppPip(context, urls.first);

    return;
  }


  final selectedUrl = await showModalBottomSheet<String>(
    context: context,
    builder: (context) {
      return SafeArea(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            const ListTile(
              title: Text(
                "Select Camera",
                style: TextStyle(fontWeight: FontWeight.bold),
              ),
            ),
            ListView.builder(
              shrinkWrap: true,
              itemCount: urls.length,
              itemBuilder: (context, index) {
                return ListTile(
                  leading: const Icon(Icons.videocam),
                  title: Text("Camera ${String.fromCharCode(65 + index)}"),
                  subtitle: Text(urls[index]),
                  onTap: () => Navigator.pop(context, urls[index]),
                );
              },
            ),
          ],
        ),
      );
    },
  );

  if (selectedUrl != null) {
    // showInAppPip(context, selectedUrl);sample
    // GlobalStreamCache.activeStreamUrl = selectedUrl; // Set the active stream URL
    showInAppPip(context, selectedUrl);
  }
}

  /// =========================================================
  /// COMMON SNACKBAR
  /// =========================================================
  static void _showSnackBar(BuildContext context, String message, Color color) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        backgroundColor: color,
        content: Text(
          message,
          style: const TextStyle(fontFamily: 'monospace', fontSize: 12),
        ),
      ),
    );
  }
}

/// =============================================================
/// FLOATING PiP WIDGET
/// =============================================================
class _FloatingPipWidget extends StatefulWidget {
  final String url;
  // sample
  // final List<String> allowedUrls;

  const _FloatingPipWidget({required this.url});
  // const _FloatingPipWidget({required this.url, required this.allowedUrls});

  @override
  State<_FloatingPipWidget> createState() => _FloatingPipWidgetState();
}

class _FloatingPipWidgetState extends State<_FloatingPipWidget> {
  /// =========================================================
  /// MK32 OPTIMIZED PiP SIZE
  /// =========================================================
  static const double pipWidth = 400;
  static const double pipHeight = 260;

  /// Initial Position
  Offset position = const Offset(20, 80);

  /// Cache controller once
 
  late String currentUrl;
  late dynamic controller;

  @override
  void initState() {
    super.initState();

    currentUrl = widget.url;
    controller = GlobalStreamCache.getController(currentUrl);
  }

  void _switchCamera(String url) {
    if (url == currentUrl) {
      debugPrint("Already on camera: $url — skipping switch");
      return;
    }

    debugPrint("Switching to camera: $url");
    debugPrint("Available players: ${GlobalStreamCache.players.keys}");

    setState(() {
      currentUrl = url;
      controller = GlobalStreamCache.getController(currentUrl);
    });
  }

  /// =========================================================
  /// DRAG HANDLER
  /// =========================================================
  void _onDragUpdate(DragUpdateDetails details) {
    final screen = MediaQuery.of(context).size;

    final newPosition = position + details.delta;

    /// Clamp to screen bounds
    final clamped = Offset(
      newPosition.dx.clamp(0, screen.width - pipWidth),
      newPosition.dy.clamp(0, screen.height - pipHeight),
    );

    /// Prevent excessive rebuilds
    if ((clamped - position).distance > 1.5) {
      setState(() {
        position = clamped;
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    return IgnorePointer(
      ignoring: false,
      child: Align(
        alignment: Alignment.topLeft,

        /// GPU-friendly movement
        child: Transform.translate(
          offset: position,

          child: GestureDetector(
            onPanUpdate: _onDragUpdate,

            child: Material(
              color: Colors.transparent,

              child: Container(
                width: pipWidth,
                height: pipHeight,

                decoration: BoxDecoration(
                  color: Colors.black,
                  borderRadius: BorderRadius.circular(8),

                  border: Border.all(color: AppTheme.cyan, width: 1.5),

                  boxShadow: const [
                    BoxShadow(
                      color: Colors.black54,
                      blurRadius: 10,
                      spreadRadius: 2,
                    ),
                  ],
                ),

                child: Stack(
                  children: [
                    /// ===================================================
                    /// VIDEO
                    /// ===================================================
                    ClipRRect(
                      borderRadius: BorderRadius.circular(6),

                      child: RepaintBoundary(
                        child: Video(
                          controller: controller,
                          controls: NoVideoControls,
                          fit: BoxFit.cover,
                        ),
                      ),
                    ),

                    /// ===================================================
                    /// CLOSE BUTTON
                    /// ===================================================
                    Positioned(
                      top: 6,
                      right: 6,

                      child: GestureDetector(
                        onTap: PipManager.hideInAppPip,

                        child: Container(
                          padding: const EdgeInsets.all(6),

                          decoration: const BoxDecoration(
                            color: Colors.black54,
                            shape: BoxShape.circle,
                          ),

                          child: const Icon(
                            Icons.close_rounded,
                            color: Colors.white,
                            size: 16,
                          ),
                        ),
                      ),
                    ),

                    /// ===================================================
                    /// NATIVE PiP BUTTON
                    /// ===================================================
                    Positioned(
                      top: 6,
                      left: 6,

                      child: GestureDetector(
                        onTap: () {
                          PipManager.hideInAppPip();

                          PipManager.enterNativePip(context);
                        },

                        child: Container(
                          padding: const EdgeInsets.all(6),

                          decoration: const BoxDecoration(
                            color: Colors.black54,
                            shape: BoxShape.circle,
                          ),

                          child: const Icon(
                            Icons.open_in_new_rounded,
                            color: Colors.white,
                            size: 16,
                          ),
                        ),
                      ),
                    ),
                      /// CAMERA BUTTONS  <-- ADD HERE
                    Positioned(
                      bottom: 8,
                      left: 8,
                      right: 8,
                      child: Row(
                        children: [
                          Expanded(
                            child: ElevatedButton(
                              onPressed: () {
                                final urls = GlobalStreamCache.players.keys.toList();
                                if (urls.isNotEmpty) {
                                  _switchCamera(urls[0]);
                                }
                                // if(widget.allowedUrls.isNotEmpty) {
                                //   _switchCamera(widget.allowedUrls[0]);
                                // }
                              },
                              child: const Text("Camera A"),
                            ),
                          ),
                          const SizedBox(width: 8),
                          Expanded(
                            child: ElevatedButton(
                              onPressed: () {
                                final urls = GlobalStreamCache.players.keys.toList();
                                if (urls.length > 1) {
                                  _switchCamera(urls[1]);
                                }
                                // if(widget.allowedUrls.isNotEmpty) {
                                //   _switchCamera(widget.allowedUrls[1]);
                                // }

                              },
                              child: const Text("Camera B"),
                              ),
                              ),
                            ],
                          ),
                        ),
                      ],
                ),
            ),
          ),
        ),
      ),
      ),
    );
  }
}

