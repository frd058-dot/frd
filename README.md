import 'dart:convert';

import 'package:flutter/cupertino.dart';
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(const PcCompatibilityApp());
}

/* =========================
   APP ROOT
   ========================= */

class PcCompatibilityApp extends StatelessWidget {
  const PcCompatibilityApp({super.key});

  @override
  Widget build(BuildContext context) {
    final cs = ColorScheme.fromSeed(seedColor: const Color(0xFF4F46E5));

    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'PC Builder',
      theme: ThemeData(
        useMaterial3: true,
        colorScheme: cs,
        scaffoldBackgroundColor: cs.surface,
        appBarTheme: AppBarTheme(
          elevation: 0,
          backgroundColor: cs.surface,
          surfaceTintColor: cs.surface,
          foregroundColor: cs.onSurface,
          centerTitle: false,
          titleTextStyle: const TextStyle(
            fontSize: 18,
            fontWeight: FontWeight.w700,
          ),
        ),
      ),
      home: const BootLoaderScreen(),
    );
  }
}

/* =========================
   BOOT LOADER
   ========================= */

class BootLoaderScreen extends StatefulWidget {
  const BootLoaderScreen({super.key});

  @override
  State<BootLoaderScreen> createState() => _BootLoaderScreenState();
}

class _BootLoaderScreenState extends State<BootLoaderScreen> {
  final _repo = PartsRepository();
  late Future<AppData> _future;

  @override
  void initState() {
    super.initState();
    _future = _repo.loadAll();
  }

  @override
  Widget build(BuildContext context) {
    return FutureBuilder<AppData>(
      future: _future,
      builder: (context, snap) {
        if (snap.hasError) {
          return Scaffold(
            appBar: AppBar(title: Text(_tr(context, tr: 'Hata', en: 'Error'))),
            body: Padding(
              padding: const EdgeInsets.all(16),
              child: Text('${snap.error}'),
            ),
          );
        }
        if (!snap.hasData) {
          return const Scaffold(
            body: Center(child: CircularProgressIndicator()),
          );
        }
        return HomeMenuScreen(data: snap.data!);
      },
    );
  }
}

/* =========================
   HOME MENU (MODERN)
   ========================= */

class HomeMenuScreen extends StatelessWidget {
  final AppData data;
  const HomeMenuScreen({super.key, required this.data});

  @override
  Widget build(BuildContext context) {
    final cs = Theme.of(context).colorScheme;

    final tiles = <_MenuTile>[
      _MenuTile(
        titleTr: 'Gaming Desktop',
        titleEn: 'Gaming Desktop',
        icon: Icons.desktop_windows_rounded,
        accent: const Color(0xFF6366F1),
        builder: (_) => PcBuilderScreen(type: BuildType.gamingDesktop, data: data),
      ),
      _MenuTile(
        titleTr: 'Gaming Notebook',
        titleEn: 'Gaming Laptop',
        icon: Icons.laptop_mac_rounded,
        accent: const Color(0xFF22C55E),
        builder: (_) => GamingLaptopConfigsScreen(data: data),
      ),
      _MenuTile(
        titleTr: 'Ofis Desktop',
        titleEn: 'Office Desktop',
        icon: Icons.computer_rounded,
        accent: const Color(0xFF0EA5E9),
        builder: (_) => PcBuilderScreen(type: BuildType.officeDesktop, data: data),
      ),
      _MenuTile(
        titleTr: 'Ofis AIO PC',
        titleEn: 'Office AIO PC',
        icon: Icons.desktop_mac_rounded,
        accent: const Color(0xFFF59E0B),
        builder: (_) => AioConfigsScreen(data: data),
      ),
      _MenuTile(
        titleTr: 'iMac AIO (Apple)',
        titleEn: 'iMac AIO (Apple)',
        icon: CupertinoIcons.desktopcomputer,
        accent: const Color(0xFF111827),
        builder: (_) => AppleCatalogScreen(
          titleTr: 'iMac AIO',
          titleEn: 'iMac AIO',
          filter: (d) => d.category.toLowerCase().contains('imac'),
          data: data,
        ),
      ),
      _MenuTile(
        titleTr: 'Apple Notebook',
        titleEn: 'Apple Notebook',
        icon: CupertinoIcons.device_laptop,
        accent: const Color(0xFF334155),
        builder: (_) => AppleCatalogScreen(
          titleTr: 'MacBook',
          titleEn: 'MacBook',
          filter: (d) => d.category.toLowerCase().contains('macbook'),
          data: data,
        ),
      ),
      _MenuTile(
        titleTr: 'Sorun Giderme',
        titleEn: 'Troubleshoot',
        icon: Icons.build_circle_rounded,
        accent: const Color(0xFFEF4444),
        builder: (_) => FixMenuScreen(data: data),
      ),
    ];

    return Scaffold(
      body: Container(
        decoration: BoxDecoration(
          gradient: LinearGradient(
            colors: [
              cs.primaryContainer.withAlpha(235),
              cs.surface,
              cs.secondaryContainer.withAlpha(235),
            ],
            begin: Alignment.topLeft,
            end: Alignment.bottomRight,
          ),
        ),
        child: Stack(
          children: [
            Positioned.fill(
              child: IgnorePointer(
                child: Center(
                  child: Opacity(
                    opacity: 0.06,
                    child: Image.asset(
                      'assets/images/techmatch.png',
                      width: 420,
                      fit: BoxFit.contain,
                    ),
                  ),
                ),
              ),
            ),
            SafeArea(
              child: Column(
                children: [
                  const SizedBox(height: 12),
                  Padding(
                    padding: const EdgeInsets.fromLTRB(16, 12, 16, 8),
                    child: _HeroHeader(
                      title: _tr(context, tr: 'PC Uyumluluk & Seçim', en: 'PC Compatibility & Builder'),
                      subtitle: _tr(
                        context,
                        tr: 'Parçaları seç, uyumluluğu gör, sistem çıktısı al.',
                        en: 'Pick parts, check compatibility, export summary.',
                      ),
                    ),
                  ),
                  Expanded(
                    child: LayoutBuilder(
                      builder: (context, constraints) {
                        final w = constraints.maxWidth;
                        final oneColumn = w < 520;

                        return GridView.builder(
                          padding: const EdgeInsets.all(16),
                          itemCount: tiles.length,
                          gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
                            crossAxisCount: oneColumn ? 1 : 2,
                            mainAxisSpacing: 14,
                            crossAxisSpacing: 14,
                            childAspectRatio: oneColumn ? 3.2 : 1.65,
                          ),
                          itemBuilder: (context, i) {
                            final tile = tiles[i];
                            return _MenuCard(
                              tile: tile,
                              onTap: () => Navigator.of(context).push(
                                MaterialPageRoute(builder: tile.builder),
                              ),
                            );
                          },
                        );
                      },
                    ),
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}

class _HeroHeader extends StatelessWidget {
  final String title;
  final String subtitle;

  const _HeroHeader({required this.title, required this.subtitle});

  @override
  Widget build(BuildContext context) {
    final cs = Theme.of(context).colorScheme;
    return Container(
      width: double.infinity,
      padding: const EdgeInsets.all(14),
      decoration: BoxDecoration(
        color: cs.surface.withAlpha(240),
        borderRadius: BorderRadius.circular(18),
        border: Border.all(color: cs.outlineVariant.withAlpha(90)),
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(title, style: const TextStyle(fontSize: 18, fontWeight: FontWeight.w800)),
          const SizedBox(height: 6),
          Text(subtitle, style: TextStyle(fontSize: 13, color: cs.onSurfaceVariant)),
        ],
      ),
    );
  }
}

class _MenuTile {
  final String titleTr;
  final String titleEn;
  final IconData icon;
  final Color accent;
  final WidgetBuilder builder;

  const _MenuTile({
    required this.titleTr,
    required this.titleEn,
    required this.icon,
    required this.accent,
    required this.builder,
  });
}

class _MenuCard extends StatelessWidget {
  final _MenuTile tile;
  final VoidCallback onTap;

  const _MenuCard({
    required this.tile,
    required this.onTap,
  });

  @override
  Widget build(BuildContext context) {
    final cs = Theme.of(context).colorScheme;

    return Card(
      elevation: 0,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(20),
        side: BorderSide(
          color: cs.outlineVariant.withAlpha(153),
        ),
      ),
      clipBehavior: Clip.antiAlias,
      child: InkWell(
        onTap: onTap,
        child: Container(
          padding: const EdgeInsets.all(16),
          decoration: BoxDecoration(
            gradient: LinearGradient(
              colors: [
                tile.accent.withAlpha(26),
                cs.surface,
              ],
              begin: Alignment.topLeft,
              end: Alignment.bottomRight,
            ),
          ),
          child: Row(
            children: [
              Container(
                padding: const EdgeInsets.all(10),
                decoration: BoxDecoration(
                  color: tile.accent.withAlpha(46),
                  borderRadius: BorderRadius.circular(18),
                ),
                child: Icon(
                  tile.icon,
                  color: tile.accent,
                  size: 28,
                ),
              ),
              const SizedBox(width: 14),
              Expanded(
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    Text(
                      _tr(context, tr: tile.titleTr, en: tile.titleEn),
                      style: const TextStyle(
                        fontSize: 16,
                        fontWeight: FontWeight.w700,
                      ),
                    ),
                    const SizedBox(height: 4),
                    Text(
                      _tr(
                        context,
                        tr: 'Hazır senaryolar ve uyumluluk yardımcısı',
                        en: 'Preset scenarios and compatibility assistant',
                      ),
                      style: TextStyle(
                        fontSize: 13,
                        color: cs.onSurfaceVariant,
                      ),
                      maxLines: 2,
                      overflow: TextOverflow.ellipsis,
                    ),
                  ],
                ),
              ),
              const SizedBox(width: 8),
              Icon(
                Icons.chevron_right_rounded,
                color: cs.onSurfaceVariant,
              ),
            ],
          ),
        ),
      ),
    );
  }
}

/* =========================
   GENERIC PICKER (BottomSheet)
   ========================= */

class _PickerFilter<T> {
  final String id;
  final String label;
  final bool Function(T) test;
  const _PickerFilter({required this.id, required this.label, required this.test});
}

Widget _pickerPill(BuildContext context, String text) {
  final cs = Theme.of(context).colorScheme;
  return Container(
    padding: const EdgeInsets.symmetric(horizontal: 10, vertical: 6),
    decoration: BoxDecoration(
      color: cs.surface.withAlpha(200),
      borderRadius: BorderRadius.circular(999),
      border: Border.all(color: cs.outlineVariant.withAlpha(90)),
    ),
    child: Text(
      text,
      style: TextStyle(
        fontSize: 12,
        fontWeight: FontWeight.w700,
        color: cs.onSurfaceVariant,
      ),
    ),
  );
}

Future<T?> _showPartPicker<T>({
  required BuildContext context,
  required String titleTr,
  required String titleEn,
  required List<T> all,
  required String Function(T) titleOf,
  required String Function(T) searchOf,
  required IconData icon,
  List<_PickerFilter<T>> filters = const [],
  String activeFilterId = 'all',
  void Function(String newFilterId)? persistFilter,
  List<String> Function(T item)? pillsOf,
  String? emptyTr,
  String? emptyEn,
}) async {
  return showModalBottomSheet<T>(
    context: context,
    isScrollControlled: true,
    useSafeArea: true,
    backgroundColor: Theme.of(context).colorScheme.surface,
    shape: const RoundedRectangleBorder(
      borderRadius: BorderRadius.vertical(top: Radius.circular(24)),
    ),
    builder: (sheetCtx) {
      String q = '';
      String fid = activeFilterId;

      bool passesFilter(T item) {
        if (filters.isEmpty) return true;
        final f = filters.firstWhere((x) => x.id == fid, orElse: () => filters.first);
        return f.test(item);
      }

      return StatefulBuilder(
        builder: (sheetCtx, setModal) {
          final ql = q.trim().toLowerCase();

          final items = all.where((item) {
            if (ql.isNotEmpty && !searchOf(item).toLowerCase().contains(ql)) return false;
            if (!passesFilter(item)) return false;
            return true;
          }).toList();

          return DraggableScrollableSheet(
            expand: false,
            initialChildSize: 0.92,
            minChildSize: 0.55,
            maxChildSize: 0.95,
            builder: (ctx, scrollCtrl) {
              return Padding(
                padding: const EdgeInsets.fromLTRB(16, 12, 16, 8),
                child: Column(
                  children: [
                    Row(
                      children: [
                        Expanded(
                          child: Text(
                            _tr(context, tr: titleTr, en: titleEn),
                            style: const TextStyle(fontSize: 16, fontWeight: FontWeight.w900),
                          ),
                        ),
                        IconButton(
                          onPressed: () => Navigator.pop(sheetCtx),
                          icon: const Icon(Icons.close_rounded),
                        ),
                      ],
                    ),
                    const SizedBox(height: 10),
                    TextField(
                      onChanged: (v) => setModal(() => q = v),
                      decoration: InputDecoration(
                        prefixIcon: const Icon(Icons.search_rounded),
                        hintText: _tr(context, tr: 'Ara...', en: 'Search...'),
                        filled: true,
                        fillColor: Theme.of(context).colorScheme.surface.withAlpha(220),
                        border: OutlineInputBorder(
                          borderRadius: BorderRadius.circular(16),
                          borderSide: BorderSide(
                            color: Theme.of(context).colorScheme.outlineVariant.withAlpha(90),
                          ),
                        ),
                        enabledBorder: OutlineInputBorder(
                          borderRadius: BorderRadius.circular(16),
                          borderSide: BorderSide(
                            color: Theme.of(context).colorScheme.outlineVariant.withAlpha(90),
                          ),
                        ),
                      ),
                    ),
                    if (filters.isNotEmpty) ...[
                      const SizedBox(height: 10),
                      Align(
                        alignment: Alignment.centerLeft,
                        child: Wrap(
                          spacing: 8,
                          runSpacing: 8,
                          children: [
                            for (final f in filters)
                              ChoiceChip(
                                label: Text(f.label),
                                selected: fid == f.id,
                                onSelected: (_) => setModal(() => fid = f.id),
                              ),
                          ],
                        ),
                      ),
                    ],
                    const SizedBox(height: 12),
                    Expanded(
                      child: items.isEmpty
                          ? Center(
                        child: Padding(
                          padding: const EdgeInsets.all(16),
                          child: Text(
                            _tr(
                              context,
                              tr: emptyTr ?? 'Kayıt bulunamadı.',
                              en: emptyEn ?? 'No items found.',
                            ),
                            textAlign: TextAlign.center,
                          ),
                        ),
                      )
                          : ListView.separated(
                        controller: scrollCtrl,
                        itemCount: items.length,
                        separatorBuilder: (_, __) => const SizedBox(height: 10),
                        itemBuilder: (_, i) {
                          final item = items[i];
                          final cs = Theme.of(context).colorScheme;
                          final pills = pillsOf?.call(item) ?? const <String>[];

                          return Card(
                            elevation: 0,
                            shape: RoundedRectangleBorder(
                              borderRadius: BorderRadius.circular(18),
                              side: BorderSide(color: cs.outlineVariant.withAlpha(90)),
                            ),
                            child: Padding(
                              padding: const EdgeInsets.all(12),
                              child: Row(
                                crossAxisAlignment: CrossAxisAlignment.start,
                                children: [
                                  Container(
                                    width: 56,
                                    height: 56,
                                    decoration: BoxDecoration(
                                      color: cs.primaryContainer.withAlpha(160),
                                      borderRadius: BorderRadius.circular(16),
                                    ),
                                    child: Icon(icon, color: cs.onPrimaryContainer),
                                  ),
                                  const SizedBox(width: 12),
                                  Expanded(
                                    child: Column(
                                      crossAxisAlignment: CrossAxisAlignment.start,
                                      children: [
                                        Text(
                                          titleOf(item),
                                          maxLines: 2,
                                          overflow: TextOverflow.ellipsis,
                                          style: const TextStyle(fontWeight: FontWeight.w900),
                                        ),
                                        if (pills.isNotEmpty) ...[
                                          const SizedBox(height: 8),
                                          Wrap(
                                            spacing: 8,
                                            runSpacing: 8,
                                            children: [
                                              for (final p in pills)
                                                if (p.trim().isNotEmpty) _pickerPill(context, p),
                                            ],
                                          ),
                                        ],
                                      ],
                                    ),
                                  ),
                                  const SizedBox(width: 8),
                                  IconButton(
                                    tooltip: _tr(context, tr: 'Seç', en: 'Select'),
                                    onPressed: () {
                                      if (persistFilter != null && filters.isNotEmpty) {
                                        persistFilter(fid);
                                      }
                                      Navigator.pop(sheetCtx, item);
                                    },
                                    icon: const Icon(Icons.add_circle_outline_rounded),
                                  ),
                                ],
                              ),
                            ),
                          );
                        },
                      ),
                    ),
                  ],
                ),
              );
            },
          );
        },
      );
    },
  );
}

/* =========================
   GAMING NOTEBOOK CONFIGS
   ========================= */

class GamingLaptopConfigsScreen extends StatelessWidget {
  final AppData data;
  const GamingLaptopConfigsScreen({super.key, required this.data});

  @override
  Widget build(BuildContext context) {
    final list = data.notebookConfigs;

    return Scaffold(
      appBar: AppBar(
        title: Text(_tr(context, tr: 'Gaming Notebook', en: 'Gaming Laptop')),
      ),
      body: list.isEmpty
          ? _placeholder(
        context,
        _tr(
          context,
          tr: 'Hazır notebook konfigürasyonları bulunamadı.\nnotebook_configs.json okunamadıysa JSON formatını kontrol et.',
          en: 'No preset configs found.\nIf notebook_configs.json cannot be read, check its JSON format.',
        ),
      )
          : ListView.separated(
        padding: const EdgeInsets.all(16),
        itemCount: list.length,
        separatorBuilder: (_, __) => const SizedBox(height: 10),
        itemBuilder: (context, i) {
          final cfg = list[i];

          return Card(
            child: ListTile(
              title: Text(cfg.name),
              subtitle: Text(cfg.specLine),
              trailing: const Icon(Icons.chevron_right),
              onTap: () => Navigator.of(context).push(
                MaterialPageRoute(builder: (_) => NotebookConfigDetailScreen(cfg: cfg)),
              ),
            ),
          );
        },
      ),
    );
  }
}

class NotebookConfigDetailScreen extends StatelessWidget {
  final NotebookConfig cfg;
  const NotebookConfigDetailScreen({super.key, required this.cfg});

  @override
  Widget build(BuildContext context) {
    final isTr = Localizations.localeOf(context).languageCode.toLowerCase() == 'tr';

    dynamic extraGet(String key) {
      final direct = cfg.extra[key];
      if (direct != null) return direct;

      final lower = key.toLowerCase();
      for (final e in cfg.extra.entries) {
        if (e.key.toString().toLowerCase() == lower) return e.value;
      }
      return null;
    }

    String extraByAliases(List<String> keys) {
      for (final k in keys) {
        final v = extraGet(k);
        if (v == null) continue;
        final s = v.toString().trim();
        if (s.isNotEmpty) return s;
      }
      return '';
    }

    String localizedExtraText({
      required List<String> trKeys,
      required List<String> enKeys,
    }) {
      final tr = extraByAliases(trKeys);
      final en = extraByAliases(enKeys);
      return isTr ? tr : en;
    }

    final usageText = localizedExtraText(
      trKeys: const ['usageTr', 'usage_tr', 'purposeTr', 'purpose_tr', 'usage', 'purpose'],
      enKeys: const ['usageEn', 'usage_en', 'purposeEn', 'purpose_en', 'usage', 'purpose'],
    ).trim();

    final notesText = localizedExtraText(
      trKeys: const ['notesTr', 'notes_tr', 'notes', 'note', 'comment'],
      enKeys: const ['notesEn', 'notes_en', 'notes', 'note', 'comment'],
    ).trim();

    const hiddenKeys = <String>{
      'usagetr',
      'usageen',
      'usage_tr',
      'usage_en',
      'purpose',
      'purposetr',
      'purposeen',
      'purpose_tr',
      'purpose_en',
      'notes',
      'notestr',
      'notesen',
      'notes_tr',
      'notes_en',
      'note',
      'comment',
    };

    String pretty(dynamic v) {
      if (v == null) return '';
      if (v is String) return v;
      try {
        return jsonEncode(v);
      } catch (_) {
        return v.toString();
      }
    }

    final extras = cfg.extra.entries
        .where((e) => !hiddenKeys.contains(e.key.toString().toLowerCase()))
        .toList()
      ..sort((a, b) => a.key.toString().compareTo(b.key.toString()));

    Widget row(String label, String value) {
      final v = value.trim();
      if (v.isEmpty) return const SizedBox.shrink();
      return Padding(
        padding: const EdgeInsets.symmetric(vertical: 6),
        child: Row(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            SizedBox(
              width: 110,
              child: Text(
                label,
                style: TextStyle(
                  fontWeight: FontWeight.w800,
                  color: Theme.of(context).colorScheme.onSurfaceVariant,
                ),
              ),
            ),
            const SizedBox(width: 10),
            Expanded(child: Text(v)),
          ],
        ),
      );
    }

    return Scaffold(
      appBar: AppBar(title: Text(cfg.name)),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          Card(
            elevation: 0,
            shape: RoundedRectangleBorder(
              borderRadius: BorderRadius.circular(16),
              side: BorderSide(color: Theme.of(context).colorScheme.outlineVariant.withAlpha(90)),
            ),
            child: Padding(
              padding: const EdgeInsets.all(14),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    _tr(context, tr: 'Sistem Özeti', en: 'System Summary'),
                    style: const TextStyle(fontSize: 15, fontWeight: FontWeight.w800),
                  ),
                  const SizedBox(height: 10),
                  if (cfg.specLine.trim().isNotEmpty) Text(cfg.specLine),
                  if (cfg.displayText.trim().isNotEmpty) ...[
                    const SizedBox(height: 6),
                    Text('${_tr(context, tr: "Ekran", en: "Display")}: ${cfg.displayText}'),
                  ],
                ],
              ),
            ),
          ),
          const SizedBox(height: 12),
          Card(
            elevation: 0,
            shape: RoundedRectangleBorder(
              borderRadius: BorderRadius.circular(16),
              side: BorderSide(color: Theme.of(context).colorScheme.outlineVariant.withAlpha(90)),
            ),
            child: Padding(
              padding: const EdgeInsets.all(14),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    _tr(context, tr: 'Donanım', en: 'Hardware'),
                    style: const TextStyle(fontSize: 15, fontWeight: FontWeight.w800),
                  ),
                  const SizedBox(height: 10),
                  row(_tr(context, tr: 'CPU', en: 'CPU'), cfg.cpu),
                  row(_tr(context, tr: 'GPU', en: 'GPU'), cfg.gpu),
                  row(_tr(context, tr: 'RAM', en: 'RAM'), cfg.ram),
                  row(_tr(context, tr: 'Depolama', en: 'Storage'), cfg.storage),
                ],
              ),
            ),
          ),
          if (usageText.isNotEmpty || notesText.isNotEmpty) ...[
            const SizedBox(height: 12),
            Card(
              elevation: 0,
              shape: RoundedRectangleBorder(
                borderRadius: BorderRadius.circular(16),
                side: BorderSide(color: Theme.of(context).colorScheme.outlineVariant.withAlpha(90)),
              ),
              child: Padding(
                padding: const EdgeInsets.all(14),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    if (usageText.isNotEmpty) ...[
                      Text(
                        _tr(context, tr: 'Kullanım Amacı', en: 'Usage'),
                        style: const TextStyle(fontSize: 15, fontWeight: FontWeight.w800),
                      ),
                      const SizedBox(height: 8),
                      Text(usageText),
                    ],
                    if (usageText.isNotEmpty && notesText.isNotEmpty) const SizedBox(height: 12),
                    if (notesText.isNotEmpty) ...[
                      Text(
                        _tr(context, tr: 'Notlar', en: 'Notes'),
                        style: const TextStyle(fontSize: 15, fontWeight: FontWeight.w800),
                      ),
                      const SizedBox(height: 8),
                      Text(notesText),
                    ],
                  ],
                ),
              ),
            ),
          ],
          if (extras.isNotEmpty) ...[
            const SizedBox(height: 12),
            Card(
              elevation: 0,
              shape: RoundedRectangleBorder(
                borderRadius: BorderRadius.circular(16),
                side: BorderSide(color: Theme.of(context).colorScheme.outlineVariant.withAlpha(90)),
              ),
              child: Padding(
                padding: const EdgeInsets.all(14),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(
                      _tr(context, tr: 'Diğer Özellikler', en: 'Other Specs'),
                      style: const TextStyle(fontSize: 15, fontWeight: FontWeight.w800),
                    ),
                    const SizedBox(height: 10),
                    for (final e in extras) row(e.key.toString(), pretty(e.value)),
                  ],
                ),
              ),
            ),
          ],
        ],
      ),
    );
  }
}

/* =========================
   AIO CONFIGS
   ========================= */

class AioConfigsScreen extends StatelessWidget {
  final AppData data;
  const AioConfigsScreen({super.key, required this.data});

  @override
  Widget build(BuildContext context) {
    final list = data.aioConfigs;

    return Scaffold(
      appBar: AppBar(
        title: Text(_tr(context, tr: 'Ofis AIO PC', en: 'Office AIO PC')),
      ),
      body: list.isEmpty
          ? _placeholder(
        context,
        _tr(
          context,
          tr: 'Hazır AIO konfigürasyonları bulunamadı.\naio_configs.json okunamadıysa JSON formatını kontrol et.',
          en: 'No preset AIO configs found.\nIf aio_configs.json cannot be read, check its JSON format.',
        ),
      )
          : ListView.separated(
        padding: const EdgeInsets.all(16),
        itemCount: list.length,
        separatorBuilder: (_, __) => const SizedBox(height: 10),
        itemBuilder: (context, i) {
          final cfg = list[i];
          return Card(
            child: ListTile(
              title: Text(cfg.name),
              subtitle: Text(cfg.specLine),
              trailing: const Icon(Icons.chevron_right),
              onTap: () => Navigator.of(context).push(
                MaterialPageRoute(builder: (_) => AioConfigDetailScreen(cfg: cfg)),
              ),
            ),
          );
        },
      ),
    );
  }
}

const Set<String> _aioHiddenKeys = {
  'usagetr',
  'usageen',
  'usage_tr',
  'usage_en',
  'purposetr',
  'purposeen',
  'purpose_tr',
  'purpose_en',
  'notestr',
  'notesen',
  'notes_tr',
  'notes_en',
};

class AioConfigDetailScreen extends StatelessWidget {
  final AioConfig cfg;
  const AioConfigDetailScreen({super.key, required this.cfg});

  @override
  Widget build(BuildContext context) {
    Widget row(String label, String value) {
      final v = value.trim();
      if (v.isEmpty) return const SizedBox.shrink();
      return Padding(
        padding: const EdgeInsets.symmetric(vertical: 6),
        child: Row(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            SizedBox(
              width: 110,
              child: Text(
                label,
                style: TextStyle(
                  fontWeight: FontWeight.w800,
                  color: Theme.of(context).colorScheme.onSurfaceVariant,
                ),
              ),
            ),
            const SizedBox(width: 10),
            Expanded(child: Text(v)),
          ],
        ),
      );
    }

    final purpose = cfg.purposeForLang(context).trim();
    final notes = cfg.notesForLang(context).trim();

    String pretty(dynamic v) {
      if (v == null) return '';
      if (v is String) return v;
      try {
        return jsonEncode(v);
      } catch (_) {
        return v.toString();
      }
    }

    final extras = cfg.extra.entries.where((e) => !_aioHiddenKeys.contains(e.key.toLowerCase())).toList()
      ..sort((a, b) => a.key.compareTo(b.key));

    return Scaffold(
      appBar: AppBar(title: Text(cfg.name)),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          Card(
            elevation: 0,
            shape: RoundedRectangleBorder(
              borderRadius: BorderRadius.circular(16),
              side: BorderSide(
                color: Theme.of(context).colorScheme.outlineVariant.withAlpha(90),
              ),
            ),
            child: Padding(
              padding: const EdgeInsets.all(14),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    _tr(context, tr: 'Sistem Özeti', en: 'System Summary'),
                    style: const TextStyle(fontSize: 15, fontWeight: FontWeight.w800),
                  ),
                  const SizedBox(height: 10),
                  if (cfg.specLine.trim().isNotEmpty) Text(cfg.specLine),
                  if (cfg.displayText.trim().isNotEmpty) ...[
                    const SizedBox(height: 6),
                    Text('${_tr(context, tr: "Ekran", en: "Display")}: ${cfg.displayText}'),
                  ],
                ],
              ),
            ),
          ),
          const SizedBox(height: 12),
          Card(
            elevation: 0,
            shape: RoundedRectangleBorder(
              borderRadius: BorderRadius.circular(16),
              side: BorderSide(
                color: Theme.of(context).colorScheme.outlineVariant.withAlpha(90),
              ),
            ),
            child: Padding(
              padding: const EdgeInsets.all(14),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  Text(
                    _tr(context, tr: 'Detaylı Özellikler', en: 'Detailed Specs'),
                    style: const TextStyle(fontSize: 15, fontWeight: FontWeight.w800),
                  ),
                  const SizedBox(height: 10),
                  row(_tr(context, tr: 'CPU', en: 'CPU'), cfg.cpu),
                  row(_tr(context, tr: 'GPU', en: 'GPU'), cfg.gpu),
                  row(_tr(context, tr: 'RAM', en: 'RAM'), cfg.ram),
                  row(_tr(context, tr: 'Depolama', en: 'Storage'), cfg.storage),
                  row(_tr(context, tr: 'Ekran', en: 'Display'), cfg.displayText),
                ],
              ),
            ),
          ),
          const SizedBox(height: 12),
          if (purpose.isNotEmpty || notes.isNotEmpty) ...[
            Card(
              elevation: 0,
              shape: RoundedRectangleBorder(
                borderRadius: BorderRadius.circular(16),
                side: BorderSide(
                  color: Theme.of(context).colorScheme.outlineVariant.withAlpha(90),
                ),
              ),
              child: Padding(
                padding: const EdgeInsets.all(14),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(
                      _tr(context, tr: 'Kullanım Amacı', en: 'Use Case'),
                      style: const TextStyle(fontSize: 15, fontWeight: FontWeight.w800),
                    ),
                    const SizedBox(height: 10),
                    if (purpose.isNotEmpty) Text(purpose),
                    if (notes.isNotEmpty) ...[
                      const SizedBox(height: 10),
                      Text(
                        _tr(context, tr: 'Notlar', en: 'Notes'),
                        style: const TextStyle(fontSize: 13, fontWeight: FontWeight.w800),
                      ),
                      const SizedBox(height: 6),
                      Text(notes),
                    ],
                  ],
                ),
              ),
            ),
          ],
          const SizedBox(height: 12),
          if (extras.isNotEmpty)
            Card(
              elevation: 0,
              shape: RoundedRectangleBorder(
                borderRadius: BorderRadius.circular(16),
                side: BorderSide(
                  color: Theme.of(context).colorScheme.outlineVariant.withAlpha(90),
                ),
              ),
              child: ExpansionTile(
                title: Text(
                  _tr(context, tr: 'Diğer Özellikler', en: 'Other Specs'),
                  style: const TextStyle(fontWeight: FontWeight.w800),
                ),
                childrenPadding: const EdgeInsets.fromLTRB(14, 0, 14, 14),
                children: [
                  for (final e in extras)
                    Padding(
                      padding: const EdgeInsets.symmetric(vertical: 6),
                      child: Row(
                        crossAxisAlignment: CrossAxisAlignment.start,
                        children: [
                          SizedBox(
                            width: 110,
                            child: Text(
                              e.key,
                              style: TextStyle(
                                fontWeight: FontWeight.w800,
                                color: Theme.of(context).colorScheme.onSurfaceVariant,
                              ),
                            ),
                          ),
                          const SizedBox(width: 10),
                          Expanded(child: Text(pretty(e.value))),
                        ],
                      ),
                    ),
                ],
              ),
            ),
        ],
      ),
    );
  }
}

/* =========================
   PC BUILDER
   ========================= */

enum BuildType { gamingDesktop, gamingNotebook, officeDesktop, officeAio }

class PcBuilderScreen extends StatefulWidget {
  final BuildType type;
  final AppData data;

  const PcBuilderScreen({super.key, required this.type, required this.data});

  @override
  State<PcBuilderScreen> createState() => _PcBuilderScreenState();
}

class _PcBuilderScreenState extends State<PcBuilderScreen> {
  Cpu? selectedCpu;
  Motherboard? selectedMotherboard;
  Ram? selectedRam;
  Storage? selectedStorage; // NVMe
  Storage? selectedSataStorage; // SATA
  Gpu? selectedGpu;
  Psu? selectedPsu;
  PcCase? selectedCase;
  Cooler? selectedCooler;

  // Gaming Desktop filtreleri (chip satırları)
  String _cpuVendor = 'all'; // all | intel | amd
  String _gpuVendor = 'all'; // all | nvidia | amd | intel

  // Screen selection
  String? selectedLaptopScreen; // 15.6 / 16 / 17.3 / 18
  String? selectedAioScreen; // 23.8 / 27

  // Motherboard RAM sekmeleri (chip)
  String _mbRamTypeFilter = 'all'; // all | ddr4 | ddr5

  // -------------------------
  // Vendor helpers
  // -------------------------
  bool _isIntelCpu(Cpu c) {
    final s = ('${c.brand} ${c.name}').toLowerCase();
    return s.contains('intel') ||
        s.contains('core') ||
        s.contains('i3') ||
        s.contains('i5') ||
        s.contains('i7') ||
        s.contains('i9') ||
        s.contains('ultra');
  }

  bool _isAmdCpu(Cpu c) {
    final s = ('${c.brand} ${c.name}').toLowerCase();
    return s.contains('amd') || s.contains('ryzen') || s.contains('threadripper');
  }

  bool _isNvidiaGpu(Gpu g) {
    final s = ('${g.brand} ${g.name}').toLowerCase();
    return s.contains('nvidia') ||
        s.contains('geforce') ||
        s.contains('rtx') ||
        s.contains('gtx') ||
        s.contains('quadro');
  }

  bool _isAmdGpu(Gpu g) {
    final s = ('${g.brand} ${g.name}').toLowerCase();
    return s.contains('amd') ||
        s.contains('radeon') ||
        s.contains('rx ') ||
        s.contains('rx-') ||
        s.contains('vega');
  }

  bool _isIntelGpu(Gpu g) {
    final s = ('${g.brand} ${g.name}').toLowerCase();
    return s.contains('intel') || s.contains('arc') || s.contains('iris') || s.contains('uhd');
  }

  List<T> _ensureSelectedInList<T>(
    T? selected,
    List<T> items,
    String Function(T) idOf,
  ) {
    if (selected == null) return items;
    final sid = idOf(selected);
    final exists = items.any((x) => idOf(x) == sid);
    return exists ? items : [selected, ...items];
  }

  // -------------------------
  // UI atoms
  // -------------------------
  Widget _pickerField({
    required BuildContext context,
    required String label,
    required String value,
    required String hint,
    required IconData icon,
    required VoidCallback onTap,
  }) {
    final cs = Theme.of(context).colorScheme;
    final showValue = value.trim().isNotEmpty;

    return Card(
      elevation: 0,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(18),
        side: BorderSide(color: cs.outlineVariant.withAlpha(90)),
      ),
      clipBehavior: Clip.antiAlias,
      child: ListTile(
        onTap: onTap,
        leading: Container(
          width: 44,
          height: 44,
          decoration: BoxDecoration(
            color: cs.primaryContainer.withAlpha(160),
            borderRadius: BorderRadius.circular(14),
          ),
          child: Icon(icon, color: cs.onPrimaryContainer),
        ),
        title: Text(label, style: const TextStyle(fontWeight: FontWeight.w900)),
        subtitle: Text(
          showValue ? value : hint,
          maxLines: 2,
          overflow: TextOverflow.ellipsis,
          style: TextStyle(color: cs.onSurfaceVariant),
        ),
        trailing: Icon(Icons.expand_more_rounded, color: cs.onSurfaceVariant),
      ),
    );
  }

  Widget _filterChipsRow({
    required List<(String id, String label)> items,
    required String selectedId,
    required void Function(String id) onPick,
  }) {
    return Padding(
      padding: const EdgeInsets.only(bottom: 10),
      child: Wrap(
        spacing: 8,
        runSpacing: 8,
        children: [
          for (final it in items)
            ChoiceChip(
              label: Text(it.$2),
              selected: selectedId == it.$1,
              onSelected: (_) => onPick(it.$1),
            ),
        ],
      ),
    );
  }

  // -------------------------
  // Pickers (BottomSheet) - ALL consistent ✅
  // -------------------------
  Future<Cpu?> _pickCpu(BuildContext context, List<Cpu> all) async {
    final isGamingDesktop = widget.type == BuildType.gamingDesktop;

    List<Cpu> base = all;
    if (isGamingDesktop) {
      if (_cpuVendor == 'intel') base = all.where(_isIntelCpu).toList();
      if (_cpuVendor == 'amd') base = all.where(_isAmdCpu).toList();
    }

    base = _ensureSelectedInList(selectedCpu, base, (x) => x.id);

    return _showPartPicker<Cpu>(
      context: context,
      titleTr: 'İşlemci Seç',
      titleEn: 'Select CPU',
      all: base,
      titleOf: (c) => c.name,
      searchOf: (c) => '${c.brand} ${c.name} ${c.socket} ${c.tdpW}',
      icon: Icons.memory_rounded,
      pillsOf: (c) => [
        if (c.brand.trim().isNotEmpty) c.brand,
        if (c.socket.trim().isNotEmpty) 'Socket ${c.socket}',
        if (c.tdpW > 0) 'TDP ${c.tdpW}W',
      ],
    );
  }

  Future<Gpu?> _pickGpu(BuildContext context, List<Gpu> all) async {
    final isGamingDesktop = widget.type == BuildType.gamingDesktop;

    List<Gpu> base = all;
    if (isGamingDesktop) {
      if (_gpuVendor == 'nvidia') base = all.where(_isNvidiaGpu).toList();
      if (_gpuVendor == 'amd') base = all.where(_isAmdGpu).toList();
      if (_gpuVendor == 'intel') base = all.where(_isIntelGpu).toList();
    }

    base = _ensureSelectedInList(selectedGpu, base, (x) => x.id);

    return _showPartPicker<Gpu>(
      context: context,
      titleTr: 'Ekran Kartı Seç',
      titleEn: 'Select GPU',
      all: base,
      titleOf: (g) => g.name,
      searchOf: (g) => '${g.brand} ${g.name} ${g.tdpW}',
      icon: Icons.grid_view_rounded,
      pillsOf: (g) => [
        if (g.brand.trim().isNotEmpty) g.brand,
        if (g.tdpW > 0) 'TDP ${g.tdpW}W',
      ],
    );
  }

  Future<Motherboard?> _pickMotherboard(BuildContext context, List<Motherboard> all) async {
    final base = _ensureSelectedInList(selectedMotherboard, all, (x) => x.id);

    return _showPartPicker<Motherboard>(
      context: context,
      titleTr: 'Anakart Seç',
      titleEn: 'Select Motherboard',
      all: base,
      titleOf: (m) => m.name,
      searchOf: (m) => '${m.name} ${m.socket} ${m.ramType} gen${m.pcieGen}',
      icon: Icons.developer_board_rounded,
      pillsOf: (m) => [
        if (m.socket.trim().isNotEmpty) 'Socket ${m.socket}',
        if (m.ramType.trim().isNotEmpty) m.ramType,
        if (m.pcieGen > 0) 'PCIe Gen${m.pcieGen}',
      ],
    );
  }

  Future<Ram?> _pickRam(BuildContext context, List<Ram> all) async {
    final base = _ensureSelectedInList(selectedRam, all, (x) => x.id);

    return _showPartPicker<Ram>(
      context: context,
      titleTr: 'RAM Seç',
      titleEn: 'Select RAM',
      all: base,
      titleOf: (r) => r.name,
      searchOf: (r) => '${r.name} ${r.type} ${r.capacityGb} ${r.speedMhz}',
      icon: Icons.storage_rounded,
      pillsOf: (r) => [
        if (r.type.trim().isNotEmpty) r.type,
        if (r.capacityGb > 0) '${r.capacityGb}GB',
        if (r.speedMhz > 0) '${r.speedMhz}MHz',
      ],
    );
  }

  String _capLabelGb(int capGb) {
    if (capGb <= 0) return '';
    return capGb >= 1024 ? '${capGb ~/ 1024}TB' : '${capGb}GB';
  }

  Future<Storage?> _pickNvme(BuildContext context, List<Storage> all) async {
    final base = _ensureSelectedInList(selectedStorage, all, (x) => x.id);

    return _showPartPicker<Storage>(
      context: context,
      titleTr: 'NVMe Seç',
      titleEn: 'Select NVMe',
      all: base,
      titleOf: (s) => s.name,
      searchOf: (s) => '${s.name} ${s.type} ${s.capacityGb} ${s.pcieGen ?? ''} ${s.readMb ?? ''}',
      icon: Icons.sd_storage_rounded,
      pillsOf: (s) => [
        if (_capLabelGb(s.capacityGb).isNotEmpty) _capLabelGb(s.capacityGb),
        if (s.pcieGen != null) 'Gen${s.pcieGen}',
        if (s.readMb != null) 'Read ${s.readMb}MB/s',
      ],
    );
  }

  Future<Storage?> _pickSata(BuildContext context, List<Storage> all) async {
    final base = _ensureSelectedInList(selectedSataStorage, all, (x) => x.id);

    return _showPartPicker<Storage>(
      context: context,
      titleTr: 'SATA SSD Seç',
      titleEn: 'Select SATA SSD',
      all: base,
      titleOf: (s) => s.name,
      searchOf: (s) => '${s.name} ${s.type} ${s.capacityGb} ${s.readMb ?? ''}',
      icon: Icons.save_rounded,
      pillsOf: (s) => [
        if (_capLabelGb(s.capacityGb).isNotEmpty) _capLabelGb(s.capacityGb),
        if (s.readMb != null) 'Read ${s.readMb}MB/s',
      ],
    );
  }

  Future<Psu?> _pickPsu(BuildContext context, List<Psu> all, int minW) async {
    final base = _ensureSelectedInList(selectedPsu, all, (x) => x.id);

    final filters = <_PickerFilter<Psu>>[
      _PickerFilter(id: 'all', label: _tr(context, tr: 'Hepsi', en: 'All'), test: (_) => true),
      _PickerFilter(
        id: 'ok',
        label: _tr(context, tr: 'Önerilen+', en: 'Recommended+'),
        test: (p) => p.watt > 0 && p.watt >= minW,
      ),
    ];

    return _showPartPicker<Psu>(
      context: context,
      titleTr: 'PSU Seç',
      titleEn: 'Select PSU',
      all: base,
      titleOf: (p) => p.name,
      searchOf: (p) => '${p.name} ${p.watt}',
      icon: Icons.power_rounded,
      filters: filters,
      activeFilterId: 'all',
      pillsOf: (p) => [
        if (p.watt > 0) '${p.watt}W',
      ],
    );
  }

  Future<PcCase?> _pickCase(BuildContext context, List<PcCase> all) async {
    final base = _ensureSelectedInList(selectedCase, all, (x) => x.id);

    return _showPartPicker<PcCase>(
      context: context,
      titleTr: 'Kasa Seç',
      titleEn: 'Select Case',
      all: base,
      titleOf: (c) => c.name,
      searchOf: (c) => c.name,
      icon: Icons.view_in_ar_rounded,
    );
  }

  Future<Cooler?> _pickCooler(BuildContext context, List<Cooler> all) async {
    final base = _ensureSelectedInList(selectedCooler, all, (x) => x.id);

    return _showPartPicker<Cooler>(
      context: context,
      titleTr: 'Soğutucu Seç',
      titleEn: 'Select Cooler',
      all: base,
      titleOf: (c) => c.name,
      searchOf: (c) => '${c.name} ${c.tdpW}',
      icon: Icons.ac_unit_rounded,
      pillsOf: (c) => [
        if (c.tdpW > 0) 'TDP ${c.tdpW}W',
      ],
    );
  }

  Future<String?> _pickScreenSize(
    BuildContext context, {
    required String titleTr,
    required String titleEn,
    required List<String> items,
    required IconData icon,
  }) {
    return _showPartPicker<String>(
      context: context,
      titleTr: titleTr,
      titleEn: titleEn,
      all: items,
      titleOf: (s) => s,
      searchOf: (s) => s,
      icon: icon,
    );
  }

  // -------------------------
  // Reset
  // -------------------------
  void resetAll() {
    setState(() {
      selectedCpu = null;
      selectedMotherboard = null;
      selectedRam = null;
      selectedStorage = null;
      selectedSataStorage = null;
      selectedGpu = null;
      selectedPsu = null;
      selectedCase = null;
      selectedCooler = null;
      selectedLaptopScreen = null;
      selectedAioScreen = null;
      _cpuVendor = 'all';
      _gpuVendor = 'all';
      _mbRamTypeFilter = 'all';
    });
  }

  @override
  Widget build(BuildContext context) {
    final cs = Theme.of(context).colorScheme;

    final title = switch (widget.type) {
      BuildType.gamingDesktop => _tr(context, tr: 'Gaming Desktop', en: 'Gaming Desktop'),
      BuildType.gamingNotebook => _tr(context, tr: 'Gaming Notebook', en: 'Gaming Laptop'),
      BuildType.officeDesktop => _tr(context, tr: 'Ofis Desktop', en: 'Office Desktop'),
      BuildType.officeAio => _tr(context, tr: 'Ofis AIO PC', en: 'Office AIO PC'),
    };

    final isLaptop = widget.type == BuildType.gamingNotebook;
    final isAio = widget.type == BuildType.officeAio;
    final isGamingDesktop = widget.type == BuildType.gamingDesktop;

    // Lists
    final cpus = isLaptop ? widget.data.laptopCpus : widget.data.cpus;
    final gpus = isLaptop ? widget.data.laptopGpus : widget.data.gpus;
    final ramsAll = isLaptop ? widget.data.laptopRams : widget.data.rams;

    final storages = widget.data.storages;
    final nvmeStorages = storages.where((s) => s.type.toUpperCase() == 'NVME').toList();
    final sataStorages = storages.where((s) => s.type.toUpperCase() == 'SATA').toList();

    final filteredNvme = nvmeStorages;
    final filteredSata = isLaptop ? <Storage>[] : sataStorages;

    final psus = widget.data.psus;
    final cases = widget.data.cases;
    final coolers = widget.data.coolers;
    final motherboards = widget.data.motherboards;

    // Motherboard socket filtering (desktop/aio)
    final filteredMotherboards = isLaptop
        ? <Motherboard>[]
        : (selectedCpu == null)
            ? motherboards
            : motherboards.where((m) => m.socket == selectedCpu!.socket).toList();

    // RAM filter by motherboard
    final filteredRams = (selectedMotherboard == null)
        ? ramsAll
        : ramsAll.where((r) => r.type == selectedMotherboard!.ramType).toList();

    // Safe selections
    selectedCpu = _safePick(selectedCpu, cpus, (x) => x.id);
    selectedGpu = _safePick(selectedGpu, gpus, (x) => x.id);
    selectedMotherboard = _safePick(selectedMotherboard, filteredMotherboards, (x) => x.id);
    selectedRam = _safePick(selectedRam, filteredRams, (x) => x.id);
    selectedStorage = _safePick(selectedStorage, filteredNvme, (x) => x.id);
    selectedSataStorage = _safePick(selectedSataStorage, filteredSata, (x) => x.id);
    selectedPsu = _safePick(selectedPsu, psus, (x) => x.id);
    selectedCase = _safePick(selectedCase, cases, (x) => x.id);
    selectedCooler = _safePick(selectedCooler, coolers, (x) => x.id);

    // Issues
    final issues = <String>[];

    if (selectedCpu != null && selectedMotherboard != null) {
      if (selectedCpu!.socket.isNotEmpty &&
          selectedMotherboard!.socket.isNotEmpty &&
          selectedCpu!.socket != selectedMotherboard!.socket) {
        issues.add(_tr(context, tr: 'CPU-Anakart soket uyumsuz', en: 'CPU-MB socket mismatch'));
      }
    }

    if (selectedRam != null && selectedMotherboard != null) {
      if (selectedRam!.type.isNotEmpty &&
          selectedMotherboard!.ramType.isNotEmpty &&
          selectedRam!.type != selectedMotherboard!.ramType) {
        issues.add(_tr(context, tr: 'RAM tipi uyumsuz', en: 'RAM type mismatch'));
      }
    }

    final recommendedPsuW = _recommendedPsuWatt(selectedCpu, selectedGpu);
    if (widget.type != BuildType.gamingNotebook) {
      if (selectedPsu != null && selectedPsu!.watt > 0 && selectedPsu!.watt < recommendedPsuW) {
        issues.add(_tr(
          context,
          tr: 'PSU gücü yetersiz (min ${recommendedPsuW}W)',
          en: 'PSU wattage is insufficient (min ${recommendedPsuW}W)',
        ));
      }
    }

    final perfWarn = _notebookPerfWarning(
      type: widget.type,
      cpuName: selectedCpu?.name ?? '',
      gpuName: selectedGpu?.name ?? '',
      context: context,
    );

    final compatible = issues.isEmpty;

    final summaryText = _buildSystemSummary(
      context,
      title: title,
      type: widget.type,
      cpu: selectedCpu,
      mb: selectedMotherboard,
      ram: selectedRam,
      storage: selectedStorage,
      sataStorage: selectedSataStorage,
      gpu: selectedGpu,
      psu: selectedPsu,
      pcCase: selectedCase,
      cooler: selectedCooler,
      laptopScreen: selectedLaptopScreen,
      aioScreen: selectedAioScreen,
      issues: issues,
      perfWarning: perfWarn,
    );

    return Scaffold(
      appBar: AppBar(
        title: Text(title),
        actions: [
          Padding(
            padding: const EdgeInsets.only(right: 10),
            child: IconButton.filledTonal(
              onPressed: resetAll,
              icon: const Icon(Icons.restart_alt_rounded),
              tooltip: _tr(context, tr: 'Sıfırla', en: 'Reset'),
            ),
          )
        ],
      ),
      body: SafeArea(
        child: ListView(
          padding: EdgeInsets.fromLTRB(
            16,
            16,
            16,
            24 + MediaQuery.of(context).padding.bottom,
          ),
          children: [
            _statusCard(
              context,
              compatible: compatible,
              issues: issues,
              perfWarning: perfWarn,
            ),
            const SizedBox(height: 14),

            // -------------------------
            // CPU (Gaming Desktop: chip row + same UI as GPU) ✅
            // -------------------------
            _sectionTitle(context, _tr(context, tr: 'CPU', en: 'CPU')),
            if (isGamingDesktop)
              _filterChipsRow(
                items: [
                  ('all', _tr(context, tr: 'Hepsi', en: 'All')),
                  ('intel', 'Intel'),
                  ('amd', 'AMD'),
                ],
                selectedId: _cpuVendor,
                onPick: (id) => setState(() => _cpuVendor = id),
              ),
            _pickerField(
              context: context,
              label: _tr(context, tr: 'CPU', en: 'CPU'),
              value: selectedCpu?.name ?? '',
              hint: _tr(context, tr: 'İşlemci seç', en: 'Select CPU'),
              icon: Icons.memory_rounded,
              onTap: () async {
                final picked = await _pickCpu(context, cpus);
                if (picked == null) return;
                setState(() {
                  selectedCpu = picked;
                  selectedMotherboard = null;
                  selectedRam = null;
                });
              },
            ),
            const SizedBox(height: 14),

            // Laptop screen (consistent picker)
            if (isLaptop) ...[
              _sectionTitle(context, _tr(context, tr: 'Ekran Boyutu', en: 'Screen Size')),
              _pickerField(
                context: context,
                label: _tr(context, tr: 'Ekran', en: 'Screen'),
                value: selectedLaptopScreen ?? '',
                hint: _tr(context, tr: 'Ekran boyutu seç', en: 'Select screen size'),
                icon: Icons.monitor_rounded,
                onTap: () async {
                  final picked = await _pickScreenSize(
                    context,
                    titleTr: 'Ekran Boyutu',
                    titleEn: 'Screen Size',
                    items: const ['15.6"', '16"', '17.3"', '18"'],
                    icon: Icons.monitor_rounded,
                  );
                  if (picked == null) return;
                  setState(() => selectedLaptopScreen = picked);
                },
              ),
              const SizedBox(height: 14),
            ],

            // AIO screen (consistent picker)
            if (isAio) ...[
              _sectionTitle(context, _tr(context, tr: 'Ekran Boyutu', en: 'Screen Size')),
              _pickerField(
                context: context,
                label: _tr(context, tr: 'Ekran', en: 'Screen'),
                value: selectedAioScreen ?? '',
                hint: _tr(context, tr: 'Ekran boyutu seç', en: 'Select screen size'),
                icon: Icons.desktop_windows_rounded,
                onTap: () async {
                  final picked = await _pickScreenSize(
                    context,
                    titleTr: 'Ekran Boyutu',
                    titleEn: 'Screen Size',
                    items: const ['23.8"', '27"'],
                    icon: Icons.desktop_windows_rounded,
                  );
                  if (picked == null) return;
                  setState(() => selectedAioScreen = picked);
                },
              ),
              const SizedBox(height: 14),
            ],

            // -------------------------
            // Motherboard (desktop/aio)
            // -------------------------
            if (widget.type != BuildType.gamingNotebook) ...[
              _sectionTitle(context, _tr(context, tr: 'Anakart', en: 'Motherboard')),

              _filterChipsRow(
                items: [
                  ('all', _tr(context, tr: 'Hepsi', en: 'All')),
                  ('ddr4', 'DDR4'),
                  ('ddr5', 'DDR5'),
                ],
                selectedId: _mbRamTypeFilter,
                onPick: (id) => setState(() => _mbRamTypeFilter = id),
              ),

              _pickerField(
                context: context,
                label: _tr(context, tr: 'Anakart', en: 'Motherboard'),
                value: selectedMotherboard?.name ?? '',
                hint: _tr(context, tr: 'Anakart seç', en: 'Select Motherboard'),
                icon: Icons.developer_board_rounded,
                onTap: () async {
                  List<Motherboard> base = filteredMotherboards;

                  if (_mbRamTypeFilter == 'ddr4') {
                    base = base.where((m) => m.ramType.toLowerCase().contains('ddr4')).toList();
                  } else if (_mbRamTypeFilter == 'ddr5') {
                    base = base.where((m) => m.ramType.toLowerCase().contains('ddr5')).toList();
                  }

                  final picked = await _pickMotherboard(context, base);
                  if (picked == null) return;

                  setState(() {
                    selectedMotherboard = picked;
                    selectedRam = null;
                  });
                },
              ),
              const SizedBox(height: 14),
            ],

            // -------------------------
            // RAM (consistent picker)
            // -------------------------
            _sectionTitle(context, _tr(context, tr: 'RAM', en: 'RAM')),
            _pickerField(
              context: context,
              label: _tr(context, tr: 'RAM', en: 'RAM'),
              value: selectedRam?.name ?? '',
              hint: _tr(context, tr: 'RAM seç', en: 'Select RAM'),
              icon: Icons.storage_rounded,
              onTap: () async {
                final picked = await _pickRam(context, filteredRams);
                if (picked == null) return;
                setState(() => selectedRam = picked);
              },
            ),
            const SizedBox(height: 14),

            // -------------------------
            // Storage (consistent picker)
            // -------------------------
            _sectionTitle(context, _tr(context, tr: 'Depolama', en: 'Storage')),
            _pickerField(
              context: context,
              label: _tr(context, tr: 'NVMe', en: 'NVMe'),
              value: selectedStorage?.name ?? '',
              hint: _tr(context, tr: 'NVMe seç', en: 'Select NVMe'),
              icon: Icons.sd_storage_rounded,
              onTap: () async {
                final picked = await _pickNvme(context, filteredNvme);
                if (picked == null) return;
                setState(() => selectedStorage = picked);
              },
            ),
            if (!isLaptop) ...[
              const SizedBox(height: 12),
              _pickerField(
                context: context,
                label: _tr(context, tr: 'SATA SSD', en: 'SATA SSD'),
                value: selectedSataStorage?.name ?? '',
                hint: _tr(context, tr: 'SATA SSD seç', en: 'Select SATA SSD'),
                icon: Icons.save_rounded,
                onTap: () async {
                  final picked = await _pickSata(context, filteredSata);
                  if (picked == null) return;
                  setState(() => selectedSataStorage = picked);
                },
              ),
            ],
            const SizedBox(height: 14),

            // -------------------------
            // GPU (Gaming Desktop: chip row + same UI as CPU) ✅
            // -------------------------
            _sectionTitle(context, _tr(context, tr: 'GPU', en: 'GPU')),
            if (isGamingDesktop)
              _filterChipsRow(
                items: [
                  ('all', _tr(context, tr: 'Hepsi', en: 'All')),
                  ('nvidia', 'NVIDIA'),
                  ('amd', 'AMD Radeon'),
                  ('intel', 'Intel'),
                ],
                selectedId: _gpuVendor,
                onPick: (id) => setState(() => _gpuVendor = id),
              ),
            _pickerField(
              context: context,
              label: _tr(context, tr: 'GPU', en: 'GPU'),
              value: selectedGpu?.name ?? '',
              hint: _tr(context, tr: 'Ekran kartı seç', en: 'Select GPU'),
              icon: Icons.grid_view_rounded,
              onTap: () async {
                final picked = await _pickGpu(context, gpus);
                if (picked == null) return;
                setState(() => selectedGpu = picked);
              },
            ),
            const SizedBox(height: 14),

            // -------------------------
            // PSU / Case / Cooler (consistent picker)
            // -------------------------
            if (widget.type != BuildType.gamingNotebook) ...[
              _sectionTitle(context, _tr(context, tr: 'PSU', en: 'PSU')),
              _pickerField(
                context: context,
                label: _tr(context, tr: 'PSU', en: 'PSU'),
                value: selectedPsu?.name ?? '',
                hint: _tr(
                  context,
                  tr: 'PSU seç (min ${recommendedPsuW}W)',
                  en: 'Select PSU (min ${recommendedPsuW}W)',
                ),
                icon: Icons.power_rounded,
                onTap: () async {
                  final picked = await _pickPsu(context, psus, recommendedPsuW);
                  if (picked == null) return;
                  setState(() => selectedPsu = picked);
                },
              ),
              const SizedBox(height: 14),

              _sectionTitle(context, _tr(context, tr: 'Kasa', en: 'Case')),
              _pickerField(
                context: context,
                label: _tr(context, tr: 'Kasa', en: 'Case'),
                value: selectedCase?.name ?? '',
                hint: _tr(context, tr: 'Kasa seç', en: 'Select Case'),
                icon: Icons.view_in_ar_rounded,
                onTap: () async {
                  final picked = await _pickCase(context, cases);
                  if (picked == null) return;
                  setState(() => selectedCase = picked);
                },
              ),
              const SizedBox(height: 14),

              _sectionTitle(context, _tr(context, tr: 'Soğutucu', en: 'Cooler')),
              _pickerField(
                context: context,
                label: _tr(context, tr: 'Soğutucu', en: 'Cooler'),
                value: selectedCooler?.name ?? '',
                hint: _tr(context, tr: 'Soğutucu seç', en: 'Select Cooler'),
                icon: Icons.ac_unit_rounded,
                onTap: () async {
                  final picked = await _pickCooler(context, coolers);
                  if (picked == null) return;
                  setState(() => selectedCooler = picked);
                },
              ),
              const SizedBox(height: 14),
            ],

            // -------------------------
            // Summary
            // -------------------------
            Card(
              elevation: 0,
              color: cs.surface.withAlpha(245),
              shape: RoundedRectangleBorder(
                borderRadius: BorderRadius.circular(16),
                side: BorderSide(color: cs.outlineVariant.withAlpha(90)),
              ),
              child: Padding(
                padding: const EdgeInsets.all(14),
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(
                      _tr(context, tr: 'Sistem Özeti', en: 'System Summary'),
                      style: const TextStyle(fontSize: 15, fontWeight: FontWeight.w800),
                    ),
                    const SizedBox(height: 8),
                    Text(
                      summaryText,
                      style: TextStyle(fontSize: 13, color: cs.onSurfaceVariant),
                    ),
                    const SizedBox(height: 12),
                    Row(
                      children: [
                        Expanded(
                          child: FilledButton.icon(
                            onPressed: () => _showSystemOutput(context, summaryText),
                            icon: const Icon(Icons.receipt_long_rounded),
                            label: Text(_tr(context, tr: 'Sistem Çıktısı', en: 'Output')),
                          ),
                        ),
                        const SizedBox(width: 10),
                        IconButton.filledTonal(
                          onPressed: () async {
                            await Clipboard.setData(ClipboardData(text: summaryText));
                            if (context.mounted) {
                              ScaffoldMessenger.of(context).showSnackBar(
                                SnackBar(
                                  content: Text(_tr(context, tr: 'Panoya kopyalandı', en: 'Copied')),
                                ),
                              );
                            }
                          },
                          icon: const Icon(Icons.copy_all_rounded),
                          tooltip: _tr(context, tr: 'Kopyala', en: 'Copy'),
                        ),
                      ],
                    ),
                  ],
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }

  Widget _statusCard(
    BuildContext context, {
    required bool compatible,
    required List<String> issues,
    required String? perfWarning,
  }) {
    final cs = Theme.of(context).colorScheme;
    final icon = compatible ? Icons.verified_rounded : Icons.error_rounded;

    final title = compatible
        ? _tr(context, tr: 'Uyumlu', en: 'Compatible')
        : _tr(context, tr: 'Uyumsuz', en: 'Incompatible');

    final details = <String>[
      if (!compatible && issues.isNotEmpty) issues.join(' • '),
      if (perfWarning != null && perfWarning.isNotEmpty) perfWarning,
    ].where((x) => x.trim().isNotEmpty).join('\n');

    return Container(
      padding: const EdgeInsets.all(14),
      decoration: BoxDecoration(
        color: cs.surface.withAlpha(245),
        borderRadius: BorderRadius.circular(16),
        border: Border.all(color: cs.outlineVariant.withAlpha(90)),
      ),
      child: Row(
        children: [
          Container(
            width: 44,
            height: 44,
            decoration: BoxDecoration(
              color: (compatible ? const Color(0xFF22C55E) : const Color(0xFFEF4444)).withAlpha(30),
              borderRadius: BorderRadius.circular(14),
            ),
            child: Icon(
              icon,
              color: compatible ? const Color(0xFF16A34A) : const Color(0xFFDC2626),
            ),
          ),
          const SizedBox(width: 12),
          Expanded(
            child: Column(crossAxisAlignment: CrossAxisAlignment.start, children: [
              Text(title, style: const TextStyle(fontSize: 15, fontWeight: FontWeight.w800)),
              const SizedBox(height: 4),
              Text(
                details.isEmpty
                    ? _tr(context, tr: 'Seçim yapınca detay görünecek.', en: 'Select parts to see details.')
                    : details,
                style: TextStyle(fontSize: 12.5, color: cs.onSurfaceVariant),
              ),
            ]),
          ),
        ],
      ),
    );
  }
}

/* =========================
   APPLE (JSON-DRIVEN CATALOG)
   ========================= */

class AppleCatalogScreen extends StatelessWidget {
  final String titleTr;
  final String titleEn;
  final bool Function(AppleDevice) filter;
  final AppData data;

  const AppleCatalogScreen({
    super.key,
    required this.titleTr,
    required this.titleEn,
    required this.filter,
    required this.data,
  });

  @override
  Widget build(BuildContext context) {
    final devices = data.appleDevices.where(filter).toList();

    return Scaffold(
      appBar: AppBar(title: Text(_tr(context, tr: titleTr, en: titleEn))),
      body: devices.isEmpty
          ? _placeholder(
        context,
        _tr(
          context,
          tr: 'apple_devices.json içinde bu kategori için cihaz yok.\nŞimdilik placeholder.',
          en: 'No devices found for this category in apple_devices.json.\nPlaceholder for now.',
        ),
      )
          : ListView.separated(
        padding: const EdgeInsets.all(16),
        itemCount: devices.length,
        separatorBuilder: (_, __) => const SizedBox(height: 10),
        itemBuilder: (context, i) {
          final d = devices[i];
          return Card(
            elevation: 0,
            shape: RoundedRectangleBorder(
              borderRadius: BorderRadius.circular(16),
              side: BorderSide(color: Theme.of(context).colorScheme.outlineVariant.withAlpha(90)),
            ),
            child: ListTile(
              title: Text(d.name, style: const TextStyle(fontWeight: FontWeight.w800)),
              subtitle: Text(d.shortSpecLine),
              trailing: const Icon(Icons.chevron_right_rounded),
              onTap: () => Navigator.of(context).push(
                MaterialPageRoute(builder: (_) => AppleDeviceDetailScreen(device: d)),
              ),
            ),
          );
        },
      ),
    );
  }
}

class AppleDeviceDetailScreen extends StatelessWidget {
  final AppleDevice device;
  const AppleDeviceDetailScreen({super.key, required this.device});

  @override
  Widget build(BuildContext context) {
    final summary = [
      '${_tr(context, tr: "Cihaz", en: "Device")}: ${device.name}',
      '${_tr(context, tr: "CPU", en: "CPU")}: ${device.cpu}',
      '${_tr(context, tr: "GPU", en: "GPU")}: ${device.gpu}',
      '${_tr(context, tr: "RAM", en: "RAM")}: ${device.ram}',
      '${_tr(context, tr: "Depolama", en: "Storage")}: ${device.storage}',
      if (device.screen.isNotEmpty) '${_tr(context, tr: "Ekran", en: "Screen")}: ${device.screen}',
      if (device.notes.isNotEmpty) '${_tr(context, tr: "Not", en: "Notes")}: ${device.notes}',
    ].join('\n');

    return Scaffold(
      appBar: AppBar(title: Text(device.name)),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Card(
          elevation: 0,
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(16),
            side: BorderSide(color: Theme.of(context).colorScheme.outlineVariant.withAlpha(90)),
          ),
          child: Padding(
            padding: const EdgeInsets.all(14),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(_tr(context, tr: 'Sistem Özeti', en: 'System Summary'), style: const TextStyle(fontSize: 15, fontWeight: FontWeight.w800)),
                const SizedBox(height: 10),
                Text(summary),
                const Spacer(),
                Row(
                  children: [
                    Expanded(
                      child: FilledButton.icon(
                        onPressed: () => _showSystemOutput(context, summary),
                        icon: const Icon(Icons.receipt_long_rounded),
                        label: Text(_tr(context, tr: 'Sistem Çıktısı', en: 'Output')),
                      ),
                    ),
                    const SizedBox(width: 10),
                    IconButton.filledTonal(
                      onPressed: () async {
                        await Clipboard.setData(ClipboardData(text: summary));
                        if (context.mounted) {
                          ScaffoldMessenger.of(context).showSnackBar(
                            SnackBar(content: Text(_tr(context, tr: 'Panoya kopyalandı', en: 'Copied'))),
                          );
                        }
                      },
                      icon: const Icon(Icons.copy_all_rounded),
                    ),
                  ],
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}

class FixMenuScreen extends StatelessWidget {
  final AppData data;
  const FixMenuScreen({super.key, required this.data});

  @override
  Widget build(BuildContext context) {
    final cs = Theme.of(context).colorScheme;
    final cats = data.fixCategories;

    return Scaffold(
      appBar: AppBar(title: Text(_tr(context, tr: 'Sorun Giderme', en: 'Troubleshoot'))),
      body: cats.isEmpty
          ? Center(
        child: Padding(
          padding: const EdgeInsets.all(16),
          child: Text(
            _tr(
              context,
              tr: 'fix_issues.json boş veya okunamadı.',
              en: 'fix_issues.json is empty or could not be loaded.',
            ),
            textAlign: TextAlign.center,
          ),
        ),
      )
          : ListView.separated(
        padding: const EdgeInsets.all(16),
        itemCount: cats.length,
        separatorBuilder: (_, __) => const SizedBox(height: 10),
        itemBuilder: (context, i) {
          final c = cats[i];
          return Card(
            elevation: 0,
            shape: RoundedRectangleBorder(
              borderRadius: BorderRadius.circular(16),
              side: BorderSide(color: cs.outlineVariant.withAlpha(90)),
            ),
            child: ListTile(
              onTap: () => Navigator.of(context).push(
                MaterialPageRoute(builder: (_) => FixCategoryScreen(category: c)),
              ),
              leading: Container(
                width: 44,
                height: 44,
                decoration: BoxDecoration(
                  color: c.accent.withAlpha(36),
                  borderRadius: BorderRadius.circular(14),
                ),
                child: Icon(c.icon, color: c.accent),
              ),
              title: Text(
                _tr(context, tr: c.titleTr, en: c.titleEn),
                style: const TextStyle(fontWeight: FontWeight.w800),
              ),
              subtitle: Text(
                _tr(
                  context,
                  tr: '${c.issues.length} başlangıç sorunu • dokun ve aç',
                  en: '${c.issues.length} starter issues • tap to open',
                ),
              ),
              trailing: const Icon(Icons.chevron_right_rounded),
            ),
          );
        },
      ),
    );
  }
}
class FixCategoryScreen extends StatelessWidget {
  final FixCategoryData category;
  const FixCategoryScreen({super.key, required this.category});

  @override
  Widget build(BuildContext context) {
    final cs = Theme.of(context).colorScheme;
    final title = _tr(context, tr: category.titleTr, en: category.titleEn);

    return Scaffold(
      appBar: AppBar(title: Text(title)),
      body: category.issues.isEmpty
          ? Center(
        child: Padding(
          padding: const EdgeInsets.all(16),
          child: Text(_tr(context, tr: 'Henüz içerik yok.', en: 'No content yet.')),
        ),
      )
          : ListView.separated(
        padding: const EdgeInsets.all(16),
        itemCount: category.issues.length,
        separatorBuilder: (_, __) => const SizedBox(height: 10),
        itemBuilder: (context, i) {
          final issue = category.issues[i];
          final issueTitle = _tr(context, tr: issue.titleTr, en: issue.titleEn);
          final symptoms = _tr(context, tr: issue.symptomsTr, en: issue.symptomsEn);

          return Card(
            elevation: 0,
            shape: RoundedRectangleBorder(
              borderRadius: BorderRadius.circular(16),
              side: BorderSide(color: cs.outlineVariant.withAlpha(90)),
            ),
            child: ListTile(
              onTap: () => Navigator.of(context).push(
                MaterialPageRoute(builder: (_) => FixIssueDetailScreen(issue: issue, accent: category.accent)),
              ),
              title: Text(issueTitle, style: const TextStyle(fontWeight: FontWeight.w800)),
              subtitle: Text(symptoms, maxLines: 2, overflow: TextOverflow.ellipsis),
              trailing: const Icon(Icons.chevron_right_rounded),
            ),
          );
        },
      ),
    );
  }
}

class FixIssueDetailScreen extends StatelessWidget {
  final FixIssueData issue;
  final Color accent;
  const FixIssueDetailScreen({super.key, required this.issue, required this.accent});

  @override
  Widget build(BuildContext context) {
    final cs = Theme.of(context).colorScheme;

    String trOrEn(String tr, String en) => _tr(context, tr: tr, en: en);

    final title = trOrEn(issue.titleTr, issue.titleEn);
    final symptoms = trOrEn(issue.symptomsTr, issue.symptomsEn);
    final quickChecks = trOrEnList(context, issue.quickChecksTr, issue.quickChecksEn);
    final stop = trOrEnList(context, issue.stopTr, issue.stopEn);

    return Scaffold(
      appBar: AppBar(title: Text(title)),
      body: ListView(
        padding: const EdgeInsets.fromLTRB(16, 16, 16, 24),
        children: [
          _fixHeaderCard(context, accent: accent, title: title, subtitle: symptoms),
          const SizedBox(height: 12),

          _fixSectionCard(
            context,
            title: trOrEn('Belirti', 'Symptoms'),
            child: Text(symptoms, style: TextStyle(color: cs.onSurfaceVariant)),
          ),

          if (quickChecks.isNotEmpty) ...[
            const SizedBox(height: 12),
            _fixSectionCard(
              context,
              title: trOrEn('Hızlı Kontroller', 'Quick checks'),
              child: _bullets(context, quickChecks),
            ),
          ],

          if (issue.stepsTr.isNotEmpty || issue.stepsEn.isNotEmpty) ...[
            const SizedBox(height: 12),
            _fixSectionCard(
              context,
              title: trOrEn('Adımlar', 'Steps'),
              child: Column(
                children: [
                  for (final step in trOrEnSteps(context, issue.stepsTr, issue.stepsEn))
                    Padding(
                      padding: const EdgeInsets.only(bottom: 10),
                      child: Card(
                        elevation: 0,
                        shape: RoundedRectangleBorder(
                          borderRadius: BorderRadius.circular(14),
                          side: BorderSide(color: cs.outlineVariant.withAlpha(90)),
                        ),
                        child: Padding(
                          padding: const EdgeInsets.all(12),
                          child: Column(
                            crossAxisAlignment: CrossAxisAlignment.start,
                            children: [
                              Text(step.title, style: const TextStyle(fontWeight: FontWeight.w900)),
                              const SizedBox(height: 6),
                              Text(step.body, style: TextStyle(color: cs.onSurfaceVariant)),
                            ],
                          ),
                        ),
                      ),
                    ),
                ],
              ),
            ),
          ],

          if (stop.isNotEmpty) ...[
            const SizedBox(height: 12),
            _fixSectionCard(
              context,
              title: trOrEn('Ne zaman durmalı?', 'When to stop'),
              child: _bullets(context, stop),
            ),
          ],
        ],
      ),
    );
  }
}

Widget _fixHeaderCard(
    BuildContext context, {
      required Color accent,
      required String title,
      required String subtitle,
    }) {
  final cs = Theme.of(context).colorScheme;
  return Container(
    padding: const EdgeInsets.all(14),
    decoration: BoxDecoration(
      color: cs.surface.withAlpha(245),
      borderRadius: BorderRadius.circular(16),
      border: Border.all(color: cs.outlineVariant.withAlpha(90)),
    ),
    child: Row(
      children: [
        Container(
          width: 44,
          height: 44,
          decoration: BoxDecoration(
            color: accent.withAlpha(36),
            borderRadius: BorderRadius.circular(14),
          ),
          child: Icon(Icons.build_circle_rounded, color: accent),
        ),
        const SizedBox(width: 12),
        Expanded(
          child: Column(crossAxisAlignment: CrossAxisAlignment.start, children: [
            Text(title, style: const TextStyle(fontSize: 15, fontWeight: FontWeight.w900)),
            const SizedBox(height: 4),
            Text(subtitle, style: TextStyle(fontSize: 12.5, color: cs.onSurfaceVariant)),
          ]),
        ),
      ],
    ),
  );
}

Widget _fixSectionCard(
    BuildContext context, {
      required String title,
      required Widget child,
    }) {
  final cs = Theme.of(context).colorScheme;
  return Card(
    elevation: 0,
    shape: RoundedRectangleBorder(
      borderRadius: BorderRadius.circular(16),
      side: BorderSide(color: cs.outlineVariant.withAlpha(90)),
    ),
    child: Padding(
      padding: const EdgeInsets.all(14),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          Text(title, style: const TextStyle(fontSize: 14, fontWeight: FontWeight.w900)),
          const SizedBox(height: 10),
          child,
        ],
      ),
    ),
  );
}

Widget _bullets(BuildContext context, List<String> items) {
  final cs = Theme.of(context).colorScheme;
  return Column(
    children: [
      for (final s in items)
        Padding(
          padding: const EdgeInsets.only(bottom: 8),
          child: Row(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Padding(
                padding: const EdgeInsets.only(top: 2),
                child: Icon(Icons.circle, size: 7, color: cs.onSurfaceVariant),
              ),
              const SizedBox(width: 10),
              Expanded(child: Text(s, style: TextStyle(color: cs.onSurfaceVariant))),
            ],
          ),
        ),
    ],
  );
}

List<_FixStepView> trOrEnSteps(BuildContext context, List<FixStepData> tr, List<FixStepData> en) {
  final isTr = Localizations.localeOf(context).languageCode.toLowerCase().startsWith('tr');

  final primary = isTr ? tr : en;
  final secondary = isTr ? en : tr;

  final picked = primary.isNotEmpty ? primary : secondary;
  final useTr = primary.isNotEmpty ? isTr : !isTr;

  return picked
      .map((s) => _FixStepView(
    title: useTr ? s.titleTr : s.titleEn,
    body: useTr ? s.bodyTr : s.bodyEn,
  ))
      .where((s) => s.title.trim().isNotEmpty || s.body.trim().isNotEmpty)
      .toList();
}
class FixStepData {
  final String titleTr;
  final String titleEn;
  final String bodyTr;
  final String bodyEn;

  const FixStepData({
    required this.titleTr,
    required this.titleEn,
    required this.bodyTr,
    required this.bodyEn,
  });

  factory FixStepData.fromJson(Map<String, dynamic> j) {
    return FixStepData(
      titleTr: _pickString(j, ['titleTr', 'title_tr'], ''),
      titleEn: _pickString(j, ['titleEn', 'title_en'], ''),
      bodyTr: _pickString(j, ['bodyTr', 'body_tr'], ''),
      bodyEn: _pickString(j, ['bodyEn', 'body_en'], ''),
    );
  }
}

/* =========================
   FIX DATA MODELS (local)
   ========================= */

String _pickString(Map<String, dynamic> j, List<String> keys, String fallback) {
  for (final k in keys) {
    final v = j[k];
    if (v == null) continue;
    final s = v.toString().trim();
    if (s.isNotEmpty) return s;
  }
  return fallback;
}

List<String> _asStringList(dynamic v) {
  if (v is List) {
    return v.map((e) => e.toString()).map((s) => s.trim()).where((s) => s.isNotEmpty).toList();
  }
  if (v is String) {
    final s = v.trim();
    return s.isEmpty ? const [] : [s];
  }
  return const [];
}

List<T> _asListOf<T>(dynamic v, T Function(Map<String, dynamic>) fromJson) {
  if (v is! List) return const [];
  return v
      .where((e) => e is Map)
      .map((e) => fromJson((e as Map).cast<String, dynamic>()))
      .toList();
}

Color _colorFrom(dynamic v, {Color fallback = const Color(0xFF6366F1)}) {
  if (v is int) return Color(v);
  if (v is String) {
    final s = v.trim();
    if (s.isEmpty) return fallback;

    // Accept "#RRGGBB" or "RRGGBB" or "0xAARRGGBB"
    String hex = s.replaceAll('#', '');
    if (hex.startsWith('0x')) {
      try {
        return Color(int.parse(hex));
      } catch (_) {
        return fallback;
      }
    }
    if (hex.length == 6) hex = 'FF$hex'; // add alpha
    if (hex.length == 8) {
      try {
        return Color(int.parse(hex, radix: 16));
      } catch (_) {
        return fallback;
      }
    }
  }
  return fallback;
}

IconData _iconFrom(dynamic v, {IconData fallback = Icons.build_circle_rounded}) {
  if (v is int) return IconData(v, fontFamily: 'MaterialIcons');
  if (v is String) {
    switch (v.trim()) {
      case 'desktop':
        return Icons.desktop_windows_rounded;
      case 'laptop':
        return Icons.laptop_mac_rounded;
      case 'computer':
        return Icons.computer_rounded;
      case 'aio':
        return Icons.desktop_mac_rounded;
      default:
        return fallback;
    }
  }
  return fallback;
}

class FixCategoryData {
  final String id;
  final String titleTr;
  final String titleEn;
  final IconData icon;
  final Color accent;
  final List<FixIssueData> issues;

  FixCategoryData({
    required this.id,
    required this.titleTr,
    required this.titleEn,
    required this.icon,
    required this.accent,
    required this.issues,
  });

  factory FixCategoryData.fromJson(Map<String, dynamic> j) {
    return FixCategoryData(
      id: _pickString(j, ['id'], ''),
      titleTr: _pickString(j, ['titleTr', 'title_tr'], ''),
      titleEn: _pickString(j, ['titleEn', 'title_en'], ''),
      icon: _iconFrom(j['icon'], fallback: Icons.build_circle_rounded),
      accent: _colorFrom(j['accent'], fallback: const Color(0xFF6366F1)),
      issues: _asListOf(j['issues'], (m) => FixIssueData.fromJson(m)),
    );
  }
}

class FixIssueData {
  final String id;
  final String titleTr;
  final String titleEn;

  final String symptomsTr;
  final String symptomsEn;

  final List<String> quickChecksTr;
  final List<String> quickChecksEn;

  final List<FixStepData> stepsTr;
  final List<FixStepData> stepsEn;

  final List<String> stopTr;
  final List<String> stopEn;

  const FixIssueData({
    required this.id,
    required this.titleTr,
    required this.titleEn,
    required this.symptomsTr,
    required this.symptomsEn,
    this.quickChecksTr = const [],
    this.quickChecksEn = const [],
    this.stepsTr = const [],
    this.stepsEn = const [],
    this.stopTr = const [],
    this.stopEn = const [],
  });

  factory FixIssueData.fromJson(Map<String, dynamic> j) {
    final qcTr = _asStringList(j['quickChecksTr'] ?? j['quickChecks_tr']);
    var qcEn = _asStringList(j['quickChecksEn'] ?? j['quickChecks_en']);

    final stTr = _asListOf(j['stepsTr'] ?? j['steps_tr'], (m) => FixStepData.fromJson(m));
    var stEn = _asListOf(j['stepsEn'] ?? j['steps_en'], (m) => FixStepData.fromJson(m));

    final spTr = _asStringList(j['stopTr'] ?? j['stop_tr']);
    var spEn = _asStringList(j['stopEn'] ?? j['stop_en']);

    // EN boşsa TR kopyala (tek dil JSON senaryosu)
    if (qcEn.isEmpty && qcTr.isNotEmpty) qcEn = List<String>.from(qcTr);
    if (stEn.isEmpty && stTr.isNotEmpty) stEn = List<FixStepData>.from(stTr);
    if (spEn.isEmpty && spTr.isNotEmpty) spEn = List<String>.from(spTr);

    return FixIssueData(
      id: _pickString(j, ['id'], ''),
      titleTr: _pickString(j, ['titleTr', 'title_tr'], ''),
      titleEn: _pickString(j, ['titleEn', 'title_en'], ''),
      symptomsTr: _pickString(j, ['symptomsTr', 'symptoms_tr'], ''),
      symptomsEn: _pickString(j, ['symptomsEn', 'symptoms_en'], ''),
      quickChecksTr: qcTr,
      quickChecksEn: qcEn,
      stepsTr: stTr,
      stepsEn: stEn,
      stopTr: spTr,
      stopEn: spEn,
    );
  }
}

class FixStepData {
  final String titleTr;
  final String titleEn;
  final String bodyTr;
  final String bodyEn;

  const FixStepData({
    required this.titleTr,
    required this.titleEn,
    required this.bodyTr,
    required this.bodyEn,
  });

  factory FixStepData.fromJson(Map<String, dynamic> j) {
    final tTr = _pickString(j, ['titleTr', 'title_tr'], '');
    final tEn = _pickString(j, ['titleEn', 'title_en'], '');
    final bTr = _pickString(j, ['bodyTr', 'body_tr'], '');
    final bEn = _pickString(j, ['bodyEn', 'body_en'], '');

    final anyTitle = _pickString(j, ['title'], '');
    final anyBody = _pickString(j, ['body'], '');

    return FixStepData(
      titleTr: tTr.isNotEmpty ? tTr : anyTitle,
      titleEn: tEn.isNotEmpty ? tEn : anyTitle,
      bodyTr: bTr.isNotEmpty ? bTr : anyBody,
      bodyEn: bEn.isNotEmpty ? bEn : anyBody,
    );
  }
}

/// JSON icon string -> IconData
IconData _fixIconFromString(String s) {
  switch (s.trim()) {
    case 'desktop_windows_rounded':
      return Icons.desktop_windows_rounded;
    case 'laptop_mac_rounded':
      return Icons.laptop_mac_rounded;
    case 'computer_rounded':
      return Icons.computer_rounded;
    case 'desktop_mac_rounded':
      return Icons.desktop_mac_rounded;
    case 'build_circle_rounded':
      return Icons.build_circle_rounded;
    default:
      return Icons.help_outline_rounded;
  }
}

/// "#RRGGBB" -> Color
Color _fixColorFromHex(String hex) {
  var h = hex.trim().replaceAll('#', '');
  if (h.length == 6) h = 'FF$h'; // alpha
  final v = int.tryParse(h, radix: 16) ?? 0xFF64748B;
  return Color(v);
}

/* =========================
   DATA LAYER (JSON)
   ========================= */

class PartsRepository {
  Future<AppData> loadAll() async {
    final results = await Future.wait([
      _loadList('assets/data/cpus.json', (j) => Cpu.fromJson(j)),
      _loadList('assets/data/motherboards.json', (j) => Motherboard.fromJson(j)),
      _loadList('assets/data/rams.json', (j) => Ram.fromJson(j)),
      _loadList('assets/data/storages.json', (j) => Storage.fromJson(j)),
      _loadList('assets/data/gpus.json', (j) => Gpu.fromJson(j)),
      _loadList('assets/data/psus.json', (j) => Psu.fromJson(j)),
      _loadList('assets/data/cases.json', (j) => PcCase.fromJson(j)),
      _loadList('assets/data/coolers.json', (j) => Cooler.fromJson(j)),
      _loadList('assets/data/apple_devices.json', (j) => AppleDevice.fromJson(j)),
      _loadList('assets/data/cpus_laptop.json', (j) => Cpu.fromJson(j)),
      _loadList('assets/data/gpus_laptop.json', (j) => Gpu.fromJson(j)),
      _loadList('assets/data/rams_laptop.json', (j) => Ram.fromJson(j)),
      _loadList('assets/data/notebook_configs.json', (j) => NotebookConfig.fromJson(j)),
      _loadList('assets/data/aig.o_configs.json', (j) => AioConfifromJson(j)),
      _loadList('assets/data/fix_issues.json', (j) => FixIssueData.fromJson(j)),
    ]);

    return AppData(
      cpus: results[0].cast<Cpu>(),
      motherboards: results[1].cast<Motherboard>(),
      rams: results[2].cast<Ram>(),
      storages: results[3].cast<Storage>(),
      gpus: results[4].cast<Gpu>(),
      psus: results[5].cast<Psu>(),
      cases: results[6].cast<PcCase>(),
      coolers: results[7].cast<Cooler>(),
      appleDevices: results[8].cast<AppleDevice>(),
      laptopCpus: results[9].cast<Cpu>(),
      laptopGpus: results[10].cast<Gpu>(),
      laptopRams: results[11].cast<Ram>(),
      notebookConfigs: results[12].cast<NotebookConfig>(),
      aioConfigs: results[13].cast<AioConfig>(),
      fixIssues: results[14].cast<FixIssueData>(),
    );
  }

 Future<List<FixCategoryData>> _loadFixIssuesFromJson() async {
  final raw = await rootBundle.loadString('assets/data/fix_issues.json');
  final decoded = jsonDecode(raw);

  if (decoded is! Map<String, dynamic>) {
    throw const FormatException('fix_issues.json root must be an object');
  }

  final cats = (decoded['categories'] as List?) ?? const [];
  return cats
      .map((e) => FixCategoryData.fromJson((e as Map).cast<String, dynamic>()))
      .toList();
}

class AppData {
  final List<Cpu> cpus;
  final List<Motherboard> motherboards;
  final List<Ram> rams;
  final List<Storage> storages;
  final List<Gpu> gpus;
  final List<Psu> psus;
  final List<PcCase> cases;
  final List<Cooler> coolers;
  final List<AppleDevice> appleDevices;

  final List<Cpu> laptopCpus;
  final List<Gpu> laptopGpus;
  final List<Ram> laptopRams;

  final List<NotebookConfig> notebookConfigs;
  final List<AioConfig> aioConfigs;

  final List<FixIssueData> fixIssues;

  const AppData({
    required this.cpus,
    required this.motherboards,
    required this.rams,
    required this.storages,
    required this.gpus,
    required this.psus,
    required this.cases,
    required this.coolers,
    required this.appleDevices,
    required this.laptopCpus,
    required this.laptopGpus,
    required this.laptopRams,
    required this.notebookConfigs,
    required this.aioConfigs,
    required this.fixCategories,
  });
}

/* =========================
   MODELS
   ========================= */

class Cpu {
  final String id;
  final String name;
  final String brand;
  final String socket;
  final int tdpW;

  const Cpu({
    required this.id,
    required this.name,
    required this.brand,
    required this.socket,
    required this.tdpW,
  });

  factory Cpu.fromJson(Map<String, dynamic> j) {
    final brand = _pickString(j, ['brand', 'vendor'], '');
    final model = _pickString(j, ['model', 'name'], '');
    final name = model.isNotEmpty ? model : _pickString(j, ['title'], 'CPU');

    return Cpu(
      id: _pickString(j, ['id'], name),
      name: name,
      brand: brand,
      socket: _pickString(j, ['socket'], ''),
      tdpW: _pickInt(j, ['tdpW', 'tdp', 'tdp_w'], 0),
    );
  }

  @override
  bool operator ==(Object other) => other is Cpu && other.id == id;

  @override
  int get hashCode => id.hashCode;
}

class Motherboard {
  final String id;
  final String name;
  final String socket;
  final String ramType;
  final int pcieGen;

  const Motherboard({
    required this.id,
    required this.name,
    required this.socket,
    required this.ramType,
    required this.pcieGen,
  });

  factory Motherboard.fromJson(Map<String, dynamic> j) {
    final brand = _pickString(j, ['brand', 'vendor'], '');
    final model = _pickString(j, ['model', 'name'], '');
    final name = model.isNotEmpty ? model : _pickString(j, ['title'], 'Motherboard');

    return Motherboard(
      id: _pickString(j, ['id'], name),
      name: (brand.isEmpty) ? name : '$brand $name',
      socket: _pickString(j, ['socket'], ''),
      ramType: _pickString(j, ['ramType', 'ram_type', 'memoryType'], ''),
      pcieGen: _pickInt(j, ['pcieGen', 'pcie_gen', 'gen'], 4),
    );
  }

  @override
  bool operator ==(Object other) => other is Motherboard && other.id == id;

  @override
  int get hashCode => id.hashCode;
}

class Ram {
  final String id;
  final String name;
  final String type;
  final int speedMhz;
  final int capacityGb;

  const Ram({
    required this.id,
    required this.name,
    required this.type,
    required this.speedMhz,
    required this.capacityGb,
  });

  factory Ram.fromJson(Map<String, dynamic> j) {
    final brand = _pickString(j, ['brand', 'vendor'], '');
    final model = _pickString(j, ['model', 'name'], '');
    final name = model.isNotEmpty ? model : _pickString(j, ['title'], 'RAM');

    return Ram(
      id: _pickString(j, ['id'], name),
      name: (brand.isEmpty) ? name : '$brand $name',
      type: _pickString(j, ['type', 'ramType', 'ram_type'], ''),
      speedMhz: _pickInt(j, ['speedMhz', 'speed', 'mhz'], 0),
      capacityGb: _pickInt(j, ['capacityGb', 'capacity', 'gb'], 0),
    );
  }

  @override
  bool operator ==(Object other) => other is Ram && other.id == id;

  @override
  int get hashCode => id.hashCode;
}

class Storage {
  final String id;
  final String name;
  final String type; // 'NVMe' veya 'SATA'
  final int capacityGb;
  final int? pcieGen; // NVMe için
  final int? readMb;

  const Storage({
    required this.id,
    required this.name,
    required this.type,
    required this.capacityGb,
    this.pcieGen,
    this.readMb,
  });

  factory Storage.fromJson(Map<String, dynamic> j) {
    final rawType = _pickString(j, ['type', 'storageType', 'kind'], '').toUpperCase();
    final isNvme = rawType.contains('NVME');

    final id = _pickString(j, ['id', 'key'], '');
    final brand = _pickString(j, ['brand', 'maker'], '');
    final model = _pickString(j, ['model', 'name', 'title'], '');
    final cap = _pickInt(j, ['capacityGb', 'capacity_gb', 'capacity', 'sizeGb', 'size_gb'], 0);
    final gen = isNvme ? _pickInt(j, ['pcieGen', 'pcie_gen', 'gen'], 4) : null;

    final read = _pickInt(j, ['readMb', 'read_mb', 'read'], 0);
    final readOrNull = read > 0 ? read : null;

    final capLabel = cap > 0 ? (cap >= 1024 ? '${cap ~/ 1024}TB' : '${cap}GB') : '';
    final genLabel = (isNvme && gen != null) ? ' Gen$gen' : '';
    final title = [
      if (brand.isNotEmpty) brand,
      if (model.isNotEmpty) model,
      if (capLabel.isNotEmpty) capLabel,
      '(${isNvme ? 'NVMe$genLabel' : 'SATA'})',
    ].join(' ').replaceAll(RegExp(r'\s+'), ' ').trim();

    final fallbackId = [
      brand,
      model,
      cap.toString(),
      isNvme ? 'nvme${gen ?? ''}' : 'sata',
    ].join('_').replaceAll(RegExp(r'[^a-zA-Z0-9_]+'), '_');

    return Storage(
      id: id.isNotEmpty ? id : fallbackId,
      name: title.isNotEmpty ? title : _pickName(j),
      type: isNvme ? 'NVMe' : 'SATA',
      capacityGb: cap,
      pcieGen: gen,
      readMb: readOrNull,
    );
  }

  @override
  bool operator ==(Object other) => other is Storage && other.id == id;

  @override
  int get hashCode => id.hashCode;
}

class NotebookConfig {
  final String id;
  final String name;

  final String cpu;
  final String gpu;
  final String ram;
  final String storage;

  final double displayInch;
  final String resolution;
  final int refreshHz;
  final String panel;

  final String purposeTr;
  final String purposeEn;
  final String notesTr;
  final String notesEn;

  final Map<String, dynamic> extra;

  const NotebookConfig({
    required this.id,
    required this.name,
    required this.cpu,
    required this.gpu,
    required this.ram,
    required this.storage,
    required this.displayInch,
    required this.resolution,
    required this.refreshHz,
    required this.panel,
    required this.purposeTr,
    required this.purposeEn,
    required this.notesTr,
    required this.notesEn,
    required this.extra,
  });

  factory NotebookConfig.fromJson(Map<String, dynamic> j) {
    final d = (j['display'] is Map) ? Map<String, dynamic>.from(j['display'] as Map) : j;

    final id = _pickString(j, ['id', 'key'], '');
    final name = _pickString(j, ['name', 'model', 'title'], _pickName(j));

    final cpu = _pickString(j, ['cpu', 'processor'], '');
    final gpu = _pickString(j, ['gpu', 'graphics'], '');
    final ram = _pickString(j, ['ram', 'memory'], '');
    final storage = _pickString(j, ['storage', 'ssd', 'disk'], '');

    final displayInch = _pickDouble(d, ['inch', 'displayInch', 'screenInch', 'size', 'diagonal'], 0);
    final resolution = _pickString(d, ['resolution', 'res'], '');
    final refreshHz = _pickInt(d, ['refreshHz', 'hz', 'refresh', 'refresh_rate'], 0);
    final panel = _pickString(d, ['panel', 'panelType', 'type'], '');

    var purposeTr = _pickString(j, ['purposeTr', 'purpose_tr'], '');
    var purposeEn = _pickString(j, ['purposeEn', 'purpose_en'], '');
    final purposeAny = _pickString(j, ['purpose', 'usage', 'summary', 'useCase'], '');
    if (purposeTr.isEmpty && purposeEn.isEmpty && purposeAny.isNotEmpty) {
      purposeTr = purposeAny;
      purposeEn = purposeAny;
    }

    var notesTr = _pickString(j, ['notesTr', 'notes_tr'], '');
    var notesEn = _pickString(j, ['notesEn', 'notes_en'], '');
    final notesAny = _pickString(j, ['notes', 'note', 'comment'], '');
    if (notesTr.isEmpty && notesEn.isEmpty && notesAny.isNotEmpty) {
      notesTr = notesAny;
      notesEn = notesAny;
    }

    return NotebookConfig(
      id: id.isNotEmpty ? id : (name.isNotEmpty ? name : 'cfg_${j.hashCode}'),
      name: name.isNotEmpty ? name : 'Notebook',
      cpu: cpu,
      gpu: gpu,
      ram: ram,
      storage: storage,
      displayInch: displayInch,
      resolution: resolution,
      refreshHz: refreshHz,
      panel: panel,
      purposeTr: purposeTr,
      purposeEn: purposeEn,
      notesTr: notesTr,
      notesEn: notesEn,
      extra: Map<String, dynamic>.from(j),
    );
  }

  String get specLine => [if (cpu.isNotEmpty) cpu, if (gpu.isNotEmpty) gpu, if (ram.isNotEmpty) ram, if (storage.isNotEmpty) storage].join(' · ');

  String get refreshHzText => refreshHz > 0 ? '${refreshHz}Hz' : '';

  String get displayText {
    final isWhole = (displayInch - displayInch.truncateToDouble()).abs() < 0.0001;
    final inchTxt = displayInch > 0 ? '${displayInch.toStringAsFixed(isWhole ? 0 : 1)}"' : '';
    final hzTxt = refreshHzText;
    final parts = <String>[
      if (inchTxt.isNotEmpty) inchTxt,
      if (resolution.isNotEmpty) resolution,
      if (hzTxt.isNotEmpty) hzTxt,
      if (panel.isNotEmpty) panel,
    ];
    return parts.join(' ').replaceAll(RegExp(r'\s+'), ' ').trim();
  }

  @override
  bool operator ==(Object other) => other is NotebookConfig && other.id == id;

  @override
  int get hashCode => id.hashCode;
}

class Gpu {
  final String id;
  final String name;
  final String brand;
  final int tdpW;

  const Gpu({
    required this.id,
    required this.name,
    required this.brand,
    required this.tdpW,
  });

  factory Gpu.fromJson(Map<String, dynamic> j) {
    final brand = _pickString(j, ['brand', 'vendor'], '');
    final model = _pickString(j, ['model', 'name'], '');
    final name = model.isNotEmpty ? model : _pickString(j, ['title'], 'GPU');

    return Gpu(
      id: _pickString(j, ['id'], name),
      name: (brand.isEmpty) ? name : '$brand $name',
      brand: brand,
      tdpW: _pickInt(j, ['tdpW', 'tdp', 'tdp_w'], 0),
    );
  }

  @override
  bool operator ==(Object other) => other is Gpu && other.id == id;

  @override
  int get hashCode => id.hashCode;
}

class Psu {
  final String id;
  final String name;
  final int watt;

  const Psu({required this.id, required this.name, required this.watt});

  factory Psu.fromJson(Map<String, dynamic> j) {
    final brand = _pickString(j, ['brand', 'vendor'], '');
    final model = _pickString(j, ['model', 'name'], '');
    final name = model.isNotEmpty ? model : _pickString(j, ['title'], 'PSU');

    return Psu(
      id: _pickString(j, ['id'], name),
      name: (brand.isEmpty) ? name : '$brand $name',
      watt: _pickInt(j, ['watt', 'w'], 0),
    );
  }

  @override
  bool operator ==(Object other) => other is Psu && other.id == id;

  @override
  int get hashCode => id.hashCode;
}

class AioConfig {
  final String id;
  final String name;

  final String cpu;
  final String gpu;
  final String ram;
  final String storage;
  final String screen;

  final String purposeTr;
  final String purposeEn;
  final String notesTr;
  final String notesEn;

  final Map<String, dynamic> extra;

  AioConfig({
    required this.id,
    required this.name,
    required this.cpu,
    required this.gpu,
    required this.ram,
    required this.storage,
    required this.screen,
    String? purpose,
    String? notes,
    String? purposeTr,
    String? purposeEn,
    String? notesTr,
    String? notesEn,
    required this.extra,
  })  : purposeTr = purposeTr ?? purpose ?? '',
        purposeEn = purposeEn ?? purpose ?? '',
        notesTr = notesTr ?? notes ?? '',
        notesEn = notesEn ?? notes ?? '';

  factory AioConfig.fromJson(Map<String, dynamic> j) {
    final id = _pickString(j, ['id', 'key'], '');
    final name = _pickString(j, ['name', 'model', 'title'], _pickName(j));

    final cpu = _pickString(j, ['cpu', 'processor'], '');
    final gpu = _pickString(j, ['gpu', 'graphics'], '');
    final ram = _pickString(j, ['ram', 'memory'], '');
    final storage = _pickString(j, ['storage', 'ssd', 'disk'], '');
    final screen = _pickString(j, ['screen', 'display', 'panel'], '');

    final purpose = _pickString(j, ['purpose', 'usage', 'summary', 'useCase'], '');
    final notes = _pickString(j, ['notes', 'note', 'comment'], '');

    final purposeTr = _pickString(j, ['purposeTr', 'purpose_tr'], '');
    final purposeEn = _pickString(j, ['purposeEn', 'purpose_en'], '');
    final notesTr = _pickString(j, ['notesTr', 'notes_tr'], '');
    final notesEn = _pickString(j, ['notesEn', 'notes_en'], '');

    final extra = Map<String, dynamic>.from(j);
    const known = <String>[
      'id',
      'key',
      'name',
      'model',
      'title',
      'cpu',
      'processor',
      'gpu',
      'graphics',
      'ram',
      'memory',
      'storage',
      'ssd',
      'disk',
      'screen',
      'display',
      'panel',
      'purpose',
      'usage',
      'summary',
      'useCase',
      'notes',
      'note',
      'comment',
      'purposeTr',
      'purpose_tr',
      'purposeEn',
      'purpose_en',
      'notesTr',
      'notes_tr',
      'notesEn',
      'notes_en',
    ];
    for (final k in known) {
      extra.remove(k);
    }

    return AioConfig(
      id: id.isNotEmpty ? id : (name.isNotEmpty ? name : 'aio_${j.hashCode}'),
      name: name.isNotEmpty ? name : 'AIO',
      cpu: cpu,
      gpu: gpu,
      ram: ram,
      storage: storage,
      screen: screen,
      purpose: purpose,
      notes: notes,
      purposeTr: purposeTr,
      purposeEn: purposeEn,
      notesTr: notesTr,
      notesEn: notesEn,
      extra: extra,
    );
  }

  String get specLine => [
    if (cpu.isNotEmpty) cpu,
    if (gpu.isNotEmpty) gpu,
    if (ram.isNotEmpty) ram,
    if (storage.isNotEmpty) storage,
    if (screen.isNotEmpty) screen,
  ].join(' · ');

  String get displayText => screen.trim();

  String purposeForLang(BuildContext context) {
    final isTr = Localizations.localeOf(context).languageCode.toLowerCase() == 'tr';
    if (isTr) return purposeTr.isNotEmpty ? purposeTr : purposeEn;
    return purposeEn.isNotEmpty ? purposeEn : purposeTr;
  }

  String notesForLang(BuildContext context) {
    final isTr = Localizations.localeOf(context).languageCode.toLowerCase() == 'tr';
    if (isTr) return notesTr.isNotEmpty ? notesTr : notesEn;
    return notesEn.isNotEmpty ? notesEn : notesTr;
  }

  @override
  bool operator ==(Object other) => other is AioConfig && other.id == id;

  @override
  int get hashCode => id.hashCode;
}

class PcCase {
  final String id;
  final String name;

  const PcCase({required this.id, required this.name});

  factory PcCase.fromJson(Map<String, dynamic> j) {
    final brand = _pickString(j, ['brand', 'vendor'], '');
    final model = _pickString(j, ['model', 'name'], '');
    final name = model.isNotEmpty ? model : _pickString(j, ['title'], 'Case');

    return PcCase(
      id: _pickString(j, ['id'], name),
      name: (brand.isEmpty) ? name : '$brand $name',
    );
  }

  @override
  bool operator ==(Object other) => other is PcCase && other.id == id;

  @override
  int get hashCode => id.hashCode;
}

class Cooler {
  final String id;
  final String name;
  final int tdpW;

  const Cooler({required this.id, required this.name, required this.tdpW});

  factory Cooler.fromJson(Map<String, dynamic> j) {
    final brand = _pickString(j, ['brand', 'vendor'], '');
    final model = _pickString(j, ['model', 'name'], '');
    final name = model.isNotEmpty ? model : _pickString(j, ['title'], 'Cooler');

    return Cooler(
      id: _pickString(j, ['id'], name),
      name: (brand.isEmpty) ? name : '$brand $name',
      tdpW: _pickInt(j, ['tdpW', 'tdp', 'tdp_w'], 0),
    );
  }

  @override
  bool operator ==(Object other) => other is Cooler && other.id == id;

  @override
  int get hashCode => id.hashCode;
}

class AppleDevice {
  final String id;
  final String category;
  final String name;
  final String cpu;
  final String gpu;
  final String ram;
  final String storage;
  final String screen;
  final String notes;

  const AppleDevice({
    required this.id,
    required this.category,
    required this.name,
    required this.cpu,
    required this.gpu,
    required this.ram,
    required this.storage,
    required this.screen,
    required this.notes,
  });

  factory AppleDevice.fromJson(Map<String, dynamic> j) {
    return AppleDevice(
      id: _pickString(j, ['id'], _pickString(j, ['name'], 'apple')),
      category: _pickString(j, ['category', 'type'], ''),
      name: _pickString(j, ['name', 'model'], 'Apple Device'),
      cpu: _pickString(j, ['cpu'], ''),
      gpu: _pickString(j, ['gpu'], ''),
      ram: _pickString(j, ['ram'], ''),
      storage: _pickString(j, ['storage'], ''),
      screen: _pickString(j, ['screen'], ''),
      notes: _pickString(j, ['notes'], ''),
    );
  }

  String get shortSpecLine {
    final parts = [
      if (cpu.isNotEmpty) cpu,
      if (ram.isNotEmpty) ram,
      if (storage.isNotEmpty) storage,
      if (screen.isNotEmpty) screen,
    ];
    return parts.join(' • ');
  }

  @override
  bool operator ==(Object other) => other is AppleDevice && other.id == id;

  @override
  int get hashCode => id.hashCode;
}

/* =========================
   HELPERS / LOGIC
   ========================= */

String _tr(BuildContext context, {required String tr, required String en}) {
  final code = Localizations.localeOf(context).languageCode.toLowerCase();
  return code.startsWith('tr') ? tr : en;
}

T? _safePick<T>(T? selected, List<T> items, String Function(T) idOf) {
  if (selected == null) return null;
  final sid = idOf(selected);
  for (final it in items) {
    if (idOf(it) == sid) return it;
  }
  return null;
}

int _recommendedPsuWatt(Cpu? cpu, Gpu? gpu) {
  final cpuW = cpu?.tdpW ?? 0;
  final gpuW = gpu?.tdpW ?? 0;

  if (cpuW == 0 && gpuW == 0) return 450;

  final base = cpuW + gpuW + 200;
  final withHeadroom = (base * 1.2).round();
  final v = withHeadroom <= 0 ? 450 : withHeadroom;
  return ((v + 49) ~/ 50) * 50;
}

String? _notebookPerfWarning({
  required BuildType type,
  required String cpuName,
  required String gpuName,
  required BuildContext context,
}) {
  if (type != BuildType.gamingNotebook) return null;

  final cpu = cpuName.toLowerCase();
  final gpu = gpuName.toLowerCase();

  final lowSeg = cpu.contains('ryzen 5') || cpu.contains(' i5') || cpu.contains('ultra 5');
  final is5070 = gpu.contains('rtx 5070');

  if (lowSeg && is5070) {
    return _tr(
      context,
      tr: 'Uyarı: Ryzen 5 / i5 / Ultra 5 segmentinde RTX 5070 seçimi pratikte verimsiz olabilir veya ürün bulunmayabilir.',
      en: 'Warning: RTX 5070 with Ryzen 5 / i5 / Ultra 5 may be inefficient or uncommon in real products.',
    );
  }
  return null;
}

String _buildSystemSummary(
    BuildContext context, {
      required String title,
      required BuildType type,
      required Cpu? cpu,
      required Motherboard? mb,
      required Ram? ram,
      required Storage? storage,
      required Storage? sataStorage,
      required Gpu? gpu,
      required Psu? psu,
      required PcCase? pcCase,
      required Cooler? cooler,
      required String? laptopScreen,
      required String? aioScreen,
      required List<String> issues,
      required String? perfWarning,
    }) {
  String line(String k, String v) => '$k: $v';

  final lines = <String>[
    line(_tr(context, tr: 'Profil', en: 'Profile'), title),
    '',
    line('CPU', cpu?.name ?? '-'),
    if (type != BuildType.gamingNotebook) line('MB', mb?.name ?? '-'),
    line('RAM', ram?.name ?? '-'),
    line(_tr(context, tr: 'NVMe', en: 'NVMe'), storage?.name ?? '-'),
    if (type != BuildType.gamingNotebook) line(_tr(context, tr: 'SATA', en: 'SATA'), sataStorage?.name ?? '-'),
    line('GPU', gpu?.name ?? '-'),
    if (type != BuildType.gamingNotebook) line('PSU', psu?.name ?? '-'),
    if (type != BuildType.gamingNotebook) line(_tr(context, tr: 'Kasa', en: 'Case'), pcCase?.name ?? '-'),
    if (type != BuildType.gamingNotebook) line(_tr(context, tr: 'Soğutucu', en: 'Cooler'), cooler?.name ?? '-'),
    if (type == BuildType.gamingNotebook) line(_tr(context, tr: 'Ekran', en: 'Screen'), laptopScreen ?? '-'),
    if (type == BuildType.officeAio) line(_tr(context, tr: 'Ekran', en: 'Screen'), aioScreen ?? '-'),
    '',
    line(
      _tr(context, tr: 'Durum', en: 'Status'),
      issues.isEmpty ? _tr(context, tr: 'Uyumlu', en: 'Compatible') : _tr(context, tr: 'Uyumsuz', en: 'Incompatible'),
    ),
    if (issues.isNotEmpty) line(_tr(context, tr: 'Sorunlar', en: 'Issues'), issues.join(' | ')),
    if (perfWarning != null && perfWarning.isNotEmpty) '',
    if (perfWarning != null && perfWarning.isNotEmpty) perfWarning,
  ];

  return lines.join('\n');
}

Future<void> _showSystemOutput(BuildContext context, String text) {
  return showDialog(
    context: context,
    builder: (_) => AlertDialog(
      title: Text(_tr(context, tr: 'Sistem Çıktısı', en: 'System Output')),
      content: SingleChildScrollView(child: Text(text)),
      actions: [
        TextButton(
          onPressed: () => Navigator.pop(context),
          child: Text(_tr(context, tr: 'Kapat', en: 'Close')),
        ),
      ],
    ),
  );
}

Widget _sectionTitle(BuildContext context, String text) {
  return Padding(
    padding: const EdgeInsets.only(bottom: 8),
    child: Text(
      text,
      style: Theme.of(context).textTheme.titleMedium?.copyWith(fontWeight: FontWeight.w800),
    ),
  );
}

Widget _placeholder(BuildContext context, String text) {
  return Center(
    child: Padding(
      padding: const EdgeInsets.all(16),
      child: Text(text, textAlign: TextAlign.center),
    ),
  );
}

String _pickString(Map<String, dynamic> j, List<String> keys, String fallback) {
  for (final k in keys) {
    final v = j[k];
    if (v == null) continue;
    final s = v.toString().trim();
    if (s.isNotEmpty) return s;
  }
  return fallback;
}

double _pickDouble(Map<String, dynamic> j, List<String> keys, double fallback) {
  for (final k in keys) {
    final v = j[k];
    if (v == null) continue;

    if (v is num) return v.toDouble();

    if (v is String) {
      final s = v.trim();
      if (s.isEmpty) continue;
      final parsed = double.tryParse(s.replaceAll(',', '.'));
      if (parsed != null) return parsed;
    }
  }
  return fallback;
}

String _pickName(Map<String, dynamic> j) {
  return _pickString(j, ['name', 'model', 'title', 'id'], 'Unknown');
}

int _pickInt(Map<String, dynamic> j, List<String> keys, int def) {
  for (final k in keys) {
    final v = j[k];
    if (v is int) return v;
    if (v is num) return v.toInt();
    if (v is String) {
      final p = int.tryParse(v.trim());
      if (p != null) return p;
    }
  }

 IconData _iconFromName(String name) {
  switch (name.trim()) {
    case 'desktop_windows_rounded':
      return Icons.desktop_windows_rounded;
    case 'laptop_mac_rounded':
      return Icons.laptop_mac_rounded;
    case 'computer_rounded':
      return Icons.computer_rounded;
    case 'desktop_mac_rounded':
      return Icons.desktop_mac_rounded;
    case 'build_circle_rounded':
      return Icons.build_circle_rounded;
    default:
      return Icons.build_circle_rounded;
  }
}

Color _colorFromHex(String input, {Color fallback = const Color(0xFF6366F1)}) {
  final s = input.trim();
  if (s.isEmpty) return fallback;

  String hex = s;
  if (hex.startsWith('#')) hex = hex.substring(1);
  if (hex.startsWith('0x') || hex.startsWith('0X')) hex = hex.substring(2);

  if (hex.length == 6) hex = 'FF$hex'; // alpha ekle
  if (hex.length != 8) return fallback;

  final v = int.tryParse(hex, radix: 16);
  if (v == null) return fallback;
  return Color(v);
}

IconData _fixIconFromString(String? raw) {
  final s = (raw ?? '').trim().toLowerCase();

  switch (s) {
    case 'desktop_windows_rounded':
    case 'desktop_windows':
      return Icons.desktop_windows_rounded;

    case 'laptop_mac_rounded':
    case 'laptop_mac':
      return Icons.laptop_mac_rounded;

    case 'computer_rounded':
    case 'computer':
      return Icons.computer_rounded;

    case 'desktop_mac_rounded':
    case 'desktop_mac':
      return Icons.desktop_mac_rounded;

    case 'build_circle_rounded':
    case 'build_circle':
      return Icons.build_circle_rounded;

    case 'developer_board_rounded':
    case 'developer_board':
      return Icons.developer_board_rounded;

    default:
      return Icons.build_circle_rounded; // fallback
  }
}

 Color _fixColorFromHex(String? raw, {Color fallback = const Color(0xFF64748B)}) {
  final s = (raw ?? '').trim();
  if (s.isEmpty) return fallback;

  // Accept: "#RRGGBB", "RRGGBB", "0xFFRRGGBB", "FFRRGGBB"
  String hex = s.toUpperCase().replaceAll('#', '');
  hex = hex.startsWith('0X') ? hex.substring(2) : hex;

  if (hex.length == 6) hex = 'FF$hex'; // add alpha
  if (hex.length != 8) return fallback;

  final v = int.tryParse(hex, radix: 16);
  if (v == null) return fallback;
  return Color(v);
}
  return def;
}
