# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

WoltModalSheet is a Flutter package that provides responsive modal sheets with multiple pages, motion animations, and scrollable content. It's designed to work across different screen sizes, adapting to bottom sheet, dialog, side sheet, or alert dialog formats based on breakpoints.

## Development Setup and Commands

This is a monorepo managed by Melos. Initial setup requires:

```bash
# Install Melos globally
dart pub global activate melos

# Bootstrap the monorepo (runs pub get for all packages)
melos bootstrap
```

### Essential Development Commands

```bash
# Format code across all packages
melos format

# Run static analysis with DCM (Dart Code Metrics)
melos analyze

# Run tests for the main package
flutter test

# Run tests across all packages
melos test
```

### Running Individual Tests

```bash
# Run a specific test file
flutter test test/wolt_modal_sheet_test.dart

# Run tests with coverage
flutter test --coverage
```

### Running Examples

```bash
# Navigate to an example directory and run
cd examples/playground
flutter run

# Available examples:
# - examples/playground - Imperative navigation demo
# - examples/playground_navigator2 - Declarative navigation (Navigator 2.0)
# - examples/coffee_maker - State management with Provider
# - examples/coffee_maker_navigator_2 - Navigator 2.0 with MVVM pattern
```

## Repository Structure

### Monorepo Organization

- **Root package**: The main `wolt_modal_sheet` package
- **packages/**: Internal utility packages
  - `wolt_di`: Dependency injection framework
  - `wolt_state_management`: State management utilities (StatefulValueNotifier, ValueState)
- **examples/**: Demonstration apps showcasing different use cases

### Core Library Structure

```
lib/src/
├── modal_type/          # Modal type implementations (bottom sheet, dialog, side sheet, alert dialog)
├── modal_page/          # Page types (Sliver, regular, non-scrolling)
├── content/             # Content layer components and animation logic
│   └── components/      # Paginating groups, main content widgets
├── widgets/             # Reusable UI components (navigation toolbar, drag detectors, barriers)
├── theme/               # Theming system and animation styles
└── utils/               # Utilities (breakpoints, keyboard handling, layout transformations)
```

## Architecture Concepts

### Modal Sheet Layering System

The modal sheet uses a z-axis layering architecture:

1. **Main Content Layer** (bottom): Contains page title, hero image, and scrollable main content
2. **Top Bar Layer**: Sits above main content with filled background; becomes sticky on scroll
3. **Navigation Bar Layer**: Transparent layer with leading/trailing nav widgets (back/close buttons)
4. **Sticky Action Bar Layer** (top): Guides user to next action; stays visible with optional gradient

### Page Types

Three distinct page implementations serve different layout needs:

- **SliverWoltModalSheetPage**: For sliver-based layouts (lists, grids). Use `mainContentSliversBuilder` to return sliver widgets
- **WoltModalSheetPage**: Simplified API for regular widgets. Wraps content in slivers internally
- **NonScrollingWoltModalSheetPage**: For flexible-height content using Column/Flex layout that shouldn't scroll

### Modal Types

Four built-in modal types adapt to screen size:

- `WoltBottomSheetType()`: Recommended for xsmall screens (< 524px)
- `WoltDialogType()`: Recommended for small/medium/large screens
- `WoltSideSheetType()`: Recommended for medium/large screens
- `WoltAlertDialogType()`: For critical alerts requiring immediate attention

Custom modal types extend `WoltModalType` and override:
- `layoutModal()`: Define constraints
- `positionModal()`: Set position offset
- `buildTransitions()`: Custom animations
- Properties like `dismissDirection`, `shapeBorder`, `barrierDismissible`, etc.

### Navigation Patterns

The package supports both imperative and declarative navigation:

**Imperative** (via `WoltModalSheet.show()`):
- Use `WoltModalSheet.of(context)` to access navigation methods
- `showNext()` / `showPrevious()` for sequential navigation
- `showAtIndex(index)` for direct page jumps
- `addPages()` / `replacePage()` / `removePage()` for stack manipulation

**Declarative** (Navigator 2.0, via `WoltModalSheet.showWithDynamicPath()`):
- Use `pageIndexNotifier` and `pageListBuilderNotifier` for state-driven navigation
- See `playground_navigator2` and `coffee_maker_navigator_2` examples

### Decorator Pattern

Two levels of decoration support state management and styling:

1. **pageContentDecorator**: Wraps only the page content (not barrier)
2. **modalDecorator**: Wraps entire modal including barrier; ideal for providing state management context (ChangeNotifierProvider, BlocProvider, etc.)

Example:
```dart
WoltModalSheet.show(
  modalDecorator: (child) => ChangeNotifierProvider<MyViewModel>(
    create: (_) => MyViewModel(),
    child: child,
  ),
  pageListBuilder: (context) => [...],
);
```

## Code Quality Requirements

### Static Analysis

- Uses DCM (Dart Code Metrics) for comprehensive static analysis
- Based on `flutter_lints` with custom DCM rules configuration
- Many DCM rules are disabled in `analysis_options.yaml`; respect existing configuration
- CI enforces DCM checks; ensure `melos analyze` passes before committing

### Testing

- Widget tests required for UI components
- Tests located in `test/` directory mirroring `lib/src/` structure
- Minimum Flutter version: 3.16.0 (used in CI)

### Formatting

- Strict formatting enforced via `dart format --set-exit-if-changed`
- CI will fail if code is not properly formatted
- Run `melos format` before committing

## Important Implementation Notes

### Page List Builder Pattern

The `pageListBuilder` function receives a `BuildContext` and should return a list of pages. This context is the modal sheet's internal context, which is important for navigation:

```dart
WoltModalSheet.show(
  pageListBuilder: (modalSheetContext) => [
    SliverWoltModalSheetPage(
      trailingNavBarWidget: IconButton(
        onPressed: Navigator.of(modalSheetContext).pop, // Use modalSheetContext
        icon: Icon(Icons.close),
      ),
      // ...
    ),
  ],
);
```

### Modal Type Responsiveness

Use `modalTypeBuilder` to return different modal types based on screen size:

```dart
modalTypeBuilder: (context) {
  final width = MediaQuery.sizeOf(context).width;
  if (width < 524) return WoltModalType.bottomSheet();
  if (width < 800) return WoltModalType.dialog();
  return WoltModalType.sideSheet();
}
```

### State Management Integration

For state that needs to persist across multiple pages in a modal, use `modalDecorator` to wrap with your state management solution. The `coffee_maker` examples demonstrate this pattern with Provider.

### CupertinoApp Support

When using CupertinoApp instead of MaterialApp, add Material localizations delegate:

```dart
CupertinoApp(
  localizationsDelegates: [DefaultMaterialLocalizations.delegate],
)
```

## Common Patterns

### Dynamic Page Updates

To update the current page based on user interaction:

```dart
WoltModalSheet.of(context).updateCurrentPage((currentPage) {
  return currentPage.copyWith(
    enableDrag: false,
    hasTopBarLayer: true,
  );
});
```

### Conditional Page Navigation

Add pages conditionally based on user input, replacing subsequent pages if user backtracks:

```dart
WoltModalSheet.of(context).addOrReplacePages([
  conditionalPage1,
  conditionalPage2,
]);
```

### In-Page Navigation Control

Use `ValueNotifier<int>` for controlling navigation from within page content:

```dart
final pageIndexNotifier = ValueNotifier(0);

WoltModalSheet.show(
  pageIndexNotifier: pageIndexNotifier,
  pageListBuilder: (context) => [
    PageOne(onNext: () => pageIndexNotifier.value++),
    PageTwo(onBack: () => pageIndexNotifier.value--),
  ],
);
```
