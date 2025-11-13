# WoltModalSheet - Complete Reference Guide

A comprehensive guide for using WoltModalSheet in Flutter applications. Use this document with any LLM when building apps that need responsive, multi-page modal sheets.

## Table of Contents

1. [Quick Start](#quick-start)
2. [Core Concepts](#core-concepts)
3. [Page Types](#page-types)
4. [Modal Types](#modal-types)
5. [Navigation](#navigation)
6. [Theming & Customization](#theming--customization)
7. [State Management](#state-management)
8. [Common Patterns](#common-patterns)
9. [API Reference](#api-reference)
10. [Troubleshooting](#troubleshooting)

---

## Quick Start

### Installation

Add to your `pubspec.yaml`:

```yaml
dependencies:
  wolt_modal_sheet: ^0.11.0
```

Run:
```bash
flutter pub add wolt_modal_sheet
```

### Basic Example

```dart
import 'package:flutter/material.dart';
import 'package:wolt_modal_sheet/wolt_modal_sheet.dart';

void showBasicModal(BuildContext context) {
  WoltModalSheet.show(
    context: context,
    pageListBuilder: (modalSheetContext) => [
      SliverWoltModalSheetPage(
        mainContentSliversBuilder: (context) => [
          SliverList.builder(
            itemCount: 20,
            itemBuilder: (context, index) => ListTile(
              title: Text('Item $index'),
              onTap: () => Navigator.of(modalSheetContext).pop(),
            ),
          ),
        ],
      ),
    ],
  );
}
```

---

## Core Concepts

### The Layer System

WoltModalSheet uses a z-axis layering architecture with 4 distinct layers:

1. **Main Content Layer** (bottom)
   - Contains page title, hero image, and scrollable main content
   - The primary content area users interact with

2. **Top Bar Layer**
   - Sits above main content with filled background
   - Shows title when user scrolls
   - Becomes sticky based on scroll position

3. **Navigation Bar Layer**
   - Transparent layer with back/close buttons
   - Always visible for navigation control
   - Overlays the top bar

4. **Sticky Action Bar Layer** (top)
   - Guides users to the next action
   - Always visible with optional gradient hint
   - Perfect for CTAs (Call-to-Actions)

### Context Matters

**CRITICAL**: The `pageListBuilder` receives the modal's internal context:

```dart
WoltModalSheet.show(
  context: context,
  pageListBuilder: (modalSheetContext) => [
    SliverWoltModalSheetPage(
      // Use modalSheetContext for navigation
      trailingNavBarWidget: IconButton(
        onPressed: () => Navigator.of(modalSheetContext).pop(), // ✅ Correct
        icon: Icon(Icons.close),
      ),
    ),
  ],
);
```

---

## Page Types

WoltModalSheet provides three page types for different use cases:

### 1. SliverWoltModalSheetPage

**Use when**: You need lists, grids, or custom scrollable layouts

**Features**:
- Lazy rendering for performance
- Custom scroll effects
- Flexible sliver-based layouts

**Example**:
```dart
SliverWoltModalSheetPage(
  pageTitle: Text('Choose an option'),
  topBarTitle: Text('Options'),
  leadingNavBarWidget: IconButton(
    icon: Icon(Icons.arrow_back),
    onPressed: () => WoltModalSheet.of(context).showPrevious(),
  ),
  trailingNavBarWidget: IconButton(
    icon: Icon(Icons.close),
    onPressed: () => Navigator.of(context).pop(),
  ),
  heroImage: Image.network(
    'https://example.com/hero.jpg',
    fit: BoxFit.cover,
  ),
  mainContentSliversBuilder: (context) => [
    SliverList.builder(
      itemCount: 100,
      itemBuilder: (context, index) => ListTile(
        title: Text('Item $index'),
      ),
    ),
  ],
  stickyActionBar: Padding(
    padding: EdgeInsets.all(16),
    child: ElevatedButton(
      onPressed: () => WoltModalSheet.of(context).showNext(),
      child: Text('Continue'),
    ),
  ),
)
```

**All Fields**:
- `id`: Unique identifier for navigation
- `pageTitle`: Widget shown above main content
- `topBarTitle`: Widget shown in sticky top bar (auto-extracted from pageTitle if null)
- `topBar`: Custom top bar widget (overrides topBarTitle)
- `heroImage`: Image shown at top of page
- `mainContentSliversBuilder`: Returns list of sliver widgets
- `leadingNavBarWidget`: Leading navigation button (usually back)
- `trailingNavBarWidget`: Trailing navigation button (usually close)
- `stickyActionBar`: Widget that stays at bottom
- `isTopBarLayerAlwaysVisible`: Keep top bar visible (default: false)
- `hasTopBarLayer`: Show/hide top bar layer (default: true)
- `hasSabGradient`: Show gradient above action bar (default: true)
- `enableDrag`: Enable drag-to-dismiss (default: true)
- `navBarHeight`: Height of navigation bar (default: 56)
- `forceMaxHeight`: Force page to use max height (default: false)
- `resizeToAvoidBottomInset`: Adjust for keyboard (default: true)

### 2. WoltModalSheetPage

**Use when**: You have simple, non-list content

**Features**:
- Simplified API - no slivers needed
- Wraps regular widgets automatically
- Best for static content

**Example**:
```dart
WoltModalSheetPage(
  pageTitle: Text('Confirmation'),
  topBarTitle: Text('Confirm'),
  trailingNavBarWidget: IconButton(
    icon: Icon(Icons.close),
    onPressed: () => Navigator.of(context).pop(),
  ),
  child: Padding(
    padding: EdgeInsets.all(16),
    child: Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        Text('Are you sure you want to proceed?'),
        SizedBox(height: 16),
        Text('This action cannot be undone.'),
      ],
    ),
  ),
  stickyActionBar: Padding(
    padding: EdgeInsets.all(16),
    child: Row(
      children: [
        Expanded(
          child: OutlinedButton(
            onPressed: () => Navigator.of(context).pop(),
            child: Text('Cancel'),
          ),
        ),
        SizedBox(width: 16),
        Expanded(
          child: ElevatedButton(
            onPressed: () {
              // Handle confirmation
              Navigator.of(context).pop();
            },
            child: Text('Confirm'),
          ),
        ),
      ],
    ),
  ),
)
```

### 3. NonScrollingWoltModalSheetPage

**Use when**: Content should NOT scroll and uses Flex layout

**Features**:
- Uses Column/Flex layout
- No scrolling behavior
- Perfect for forms, dialogs, fixed-height content

**Example**:
```dart
NonScrollingWoltModalSheetPage(
  pageTitle: Text('Select Payment Method'),
  topBarTitle: Text('Payment'),
  child: Column(
    mainAxisSize: MainAxisSize.min,
    children: [
      RadioListTile(
        title: Text('Credit Card'),
        value: 'card',
        groupValue: selectedPayment,
        onChanged: (value) => setState(() => selectedPayment = value),
      ),
      RadioListTile(
        title: Text('PayPal'),
        value: 'paypal',
        groupValue: selectedPayment,
        onChanged: (value) => setState(() => selectedPayment = value),
      ),
      RadioListTile(
        title: Text('Apple Pay'),
        value: 'apple',
        groupValue: selectedPayment,
        onChanged: (value) => setState(() => selectedPayment = value),
      ),
    ],
  ),
  stickyActionBar: Padding(
    padding: EdgeInsets.all(16),
    child: ElevatedButton(
      onPressed: () => WoltModalSheet.of(context).showNext(),
      child: Text('Continue'),
    ),
  ),
)
```

---

## Modal Types

### Built-in Types

WoltModalSheet provides 4 responsive modal types:

#### 1. Bottom Sheet
```dart
WoltModalType.bottomSheet()
// OR
WoltBottomSheetType()
```
- **Recommended for**: xsmall screens (< 524px)
- Slides up from bottom
- Dismissible by dragging down
- Default drag handle

#### 2. Dialog
```dart
WoltModalType.dialog()
// OR
WoltDialogType()
```
- **Recommended for**: small/medium/large screens (≥ 524px)
- Centered on screen
- Fixed width (512px by default)
- Barrier dismissible

#### 3. Side Sheet
```dart
WoltModalType.sideSheet()
// OR
WoltSideSheetType()
```
- **Recommended for**: medium/large screens (≥ 768px)
- Slides in from right (or left for RTL)
- Full height
- Dismissible by dragging horizontally

#### 4. Alert Dialog
```dart
WoltModalType.alertDialog()
// OR
WoltAlertDialogType()
```
- **For critical alerts** requiring immediate attention
- Smaller, centered
- Not barrier dismissible by default
- No drag-to-dismiss

### Responsive Modal Types

**Best practice**: Use `modalTypeBuilder` to adapt to screen size:

```dart
WoltModalSheet.show(
  context: context,
  modalTypeBuilder: (context) {
    final width = MediaQuery.sizeOf(context).width;
    if (width < 524) {
      return WoltModalType.bottomSheet();
    } else if (width < 768) {
      return WoltModalType.dialog();
    } else {
      return WoltModalType.sideSheet();
    }
  },
  pageListBuilder: (context) => [...],
);
```

### Customizing Modal Types

#### Method 1: Using copyWith
```dart
WoltModalSheet.show(
  context: context,
  modalTypeBuilder: (_) => WoltModalType.bottomSheet().copyWith(
    barrierDismissible: false,
    showDragHandle: false,
  ),
  pageListBuilder: (context) => [...],
);
```

#### Method 2: Extending Classes
```dart
class MyCustomBottomSheet extends WoltBottomSheetType {
  const MyCustomBottomSheet()
      : super(
          shapeBorder: RoundedRectangleBorder(
            borderRadius: BorderRadius.vertical(top: Radius.circular(28)),
          ),
          showDragHandle: false,
          barrierDismissible: false,
        );
}

// Use it:
modalTypeBuilder: (_) => MyCustomBottomSheet()
```

#### Method 3: Creating Custom Modal Type
```dart
class TopNotificationSheet extends WoltModalType {
  const TopNotificationSheet()
      : super(
          dismissDirection: WoltModalDismissDirection.up,
          showDragHandle: false,
          closeProgressThreshold: 0.8,
        );

  @override
  BoxConstraints layoutModal(Size availableSize) {
    return BoxConstraints(
      minWidth: 312,
      maxWidth: 312,
      minHeight: 0,
      maxHeight: availableSize.height * 0.6,
    );
  }

  @override
  Offset positionModal(
    Size availableSize,
    Size modalContentSize,
    TextDirection textDirection,
  ) {
    final xOffset = (availableSize.width - modalContentSize.width) / 2;
    return Offset(xOffset, 32);
  }

  @override
  String routeLabel(BuildContext context) {
    return MaterialLocalizations.of(context).dialogLabel;
  }

  @override
  Widget buildTransitions(
    BuildContext context,
    Animation<double> animation,
    Animation<double> secondaryAnimation,
    Widget child,
  ) {
    return SlideTransition(
      position: animation.drive(
        Tween(
          begin: Offset(0.0, -1.0),
          end: Offset.zero,
        ).chain(CurveTween(curve: Curves.easeOutQuad)),
      ),
      child: FadeTransition(
        opacity: animation,
        child: child,
      ),
    );
  }
}
```

---

## Navigation

### Imperative Navigation (Navigator 1.0)

Use `WoltModalSheet.show()` for imperative navigation:

#### Basic Navigation Methods

```dart
// Access navigation methods
final woltSheet = WoltModalSheet.of(context);

// Navigate to next page
woltSheet.showNext(); // Returns bool (true if moved)

// Navigate to previous page
woltSheet.showPrevious(); // Returns bool

// Jump to specific index
woltSheet.showAtIndex(2); // Returns bool

// Navigate by page ID
woltSheet.showPageWithId(pageId); // Returns bool

// Navigate by page type
woltSheet.showPage<MyCustomPage>(); // Returns bool

// Navigate with filter
woltSheet.showPage<MyCustomPage>(
  where: (page) => page.someProperty == "value",
);
```

#### Stack Manipulation

```dart
final woltSheet = WoltModalSheet.of(context);

// Add pages to end
woltSheet.addPages([page1, page2, page3]);
woltSheet.addPage(newPage);

// Push pages (add and navigate to first new page)
woltSheet.pushPages([page1, page2]);
woltSheet.pushPage(newPage);

// Replace specific page
woltSheet.replacePage(pageId, newPage);

// Replace current page (with animation)
woltSheet.replaceCurrentPage(newPage);

// Update current page properties
woltSheet.updateCurrentPage((currentPage) {
  return currentPage.copyWith(
    enableDrag: false,
    hasTopBarLayer: true,
  );
});

// Remove specific page
woltSheet.removePage(pageId);

// Pop last page
woltSheet.popPage(); // Returns bool

// Add or replace pages after current
// If current is last: appends new pages
// Otherwise: replaces all pages after current
woltSheet.addOrReplacePages([page1, page2]);
woltSheet.addOrReplacePage(newPage);
```

#### Multi-Page Example

```dart
void showMultiPageModal(BuildContext context) {
  WoltModalSheet.show(
    context: context,
    pageListBuilder: (modalSheetContext) => [
      // Page 1
      WoltModalSheetPage(
        pageTitle: Text('Step 1: Choose Category'),
        child: CategorySelector(),
        stickyActionBar: ElevatedButton(
          onPressed: () => WoltModalSheet.of(modalSheetContext).showNext(),
          child: Text('Next'),
        ),
      ),
      // Page 2
      WoltModalSheetPage(
        pageTitle: Text('Step 2: Enter Details'),
        leadingNavBarWidget: IconButton(
          icon: Icon(Icons.arrow_back),
          onPressed: () => WoltModalSheet.of(modalSheetContext).showPrevious(),
        ),
        child: DetailsForm(),
        stickyActionBar: ElevatedButton(
          onPressed: () => WoltModalSheet.of(modalSheetContext).showNext(),
          child: Text('Next'),
        ),
      ),
      // Page 3
      WoltModalSheetPage(
        pageTitle: Text('Step 3: Confirm'),
        leadingNavBarWidget: IconButton(
          icon: Icon(Icons.arrow_back),
          onPressed: () => WoltModalSheet.of(modalSheetContext).showPrevious(),
        ),
        child: ConfirmationView(),
        stickyActionBar: ElevatedButton(
          onPressed: () {
            // Submit and close
            submitData();
            Navigator.of(modalSheetContext).pop();
          },
          child: Text('Submit'),
        ),
      ),
    ],
  );
}
```

### Declarative Navigation (Navigator 2.0)

Use `WoltModalSheet.showWithDynamicPath()` with ValueNotifiers:

```dart
class MyWidget extends StatefulWidget {
  @override
  _MyWidgetState createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  final pageIndexNotifier = ValueNotifier<int>(0);
  late final pageListBuilderNotifier = ValueNotifier<WoltModalSheetPageListBuilder>(
    (context) => _buildPages(context),
  );

  List<SliverWoltModalSheetPage> _buildPages(BuildContext context) {
    return [
      WoltModalSheetPage(
        pageTitle: Text('Page 1'),
        child: Column(
          children: [
            Text('First page content'),
            ElevatedButton(
              onPressed: () => pageIndexNotifier.value = 1,
              child: Text('Go to Page 2'),
            ),
          ],
        ),
      ),
      WoltModalSheetPage(
        pageTitle: Text('Page 2'),
        child: Column(
          children: [
            Text('Second page content'),
            ElevatedButton(
              onPressed: () => pageIndexNotifier.value = 0,
              child: Text('Back to Page 1'),
            ),
          ],
        ),
      ),
    ];
  }

  void _showModal() {
    WoltModalSheet.showWithDynamicPath(
      context: context,
      pageIndexNotifier: pageIndexNotifier,
      pageListBuilderNotifier: pageListBuilderNotifier,
    );
  }

  @override
  void dispose() {
    pageIndexNotifier.dispose();
    pageListBuilderNotifier.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: _showModal,
      child: Text('Show Modal'),
    );
  }
}
```

---

## Theming & Customization

### Using Theme Extension

Add `WoltModalSheetThemeData` to your app theme:

```dart
MaterialApp(
  theme: ThemeData.light().copyWith(
    extensions: [
      WoltModalSheetThemeData(
        backgroundColor: Colors.white,
        modalBarrierColor: Colors.black54,
        modalElevation: 8.0,
        heroImageHeight: 200.0,
        topBarShadowColor: Colors.grey.shade300,
        topBarElevation: 4.0,
        dragHandleColor: Colors.grey,
        dragHandleSize: Size(48, 4),
        showDragHandle: true,
        enableDrag: true,
        hasSabGradient: true,
        sabGradientColor: Colors.white,
        navBarHeight: 56.0,
        mainContentScrollPhysics: ClampingScrollPhysics(),
      ),
    ],
  ),
  darkTheme: ThemeData.dark().copyWith(
    extensions: [
      WoltModalSheetThemeData(
        backgroundColor: Colors.grey.shade900,
        modalBarrierColor: Colors.white12,
        sabGradientColor: Colors.grey.shade900,
      ),
    ],
  ),
);
```

### Theme Properties

All customizable properties in `WoltModalSheetThemeData`:

- `backgroundColor`: Modal sheet background color
- `modalElevation`: Elevation of the modal (default: 2.0)
- `modalBarrierColor`: Color of the barrier behind modal
- `showDragHandle`: Show/hide drag handle
- `dragHandleColor`: Color of drag handle
- `dragHandleSize`: Size of drag handle
- `enableDrag`: Enable/disable dragging
- `topBarShadowColor`: Shadow color for top bar
- `topBarElevation`: Elevation of top bar
- `heroImageHeight`: Default height for hero images
- `hasSabGradient`: Show gradient above sticky action bar
- `sabGradientColor`: Color for sticky action bar gradient
- `sabGradientHeight`: Height of gradient
- `navBarHeight`: Height of navigation bar
- `hasTopBarLayer`: Show/hide top bar layer
- `isTopBarLayerAlwaysVisible`: Keep top bar always visible
- `surfaceTintColor`: Material 3 surface tint color
- `shadowColor`: Shadow color for modal
- `clipBehavior`: Clip behavior for modal
- `mainContentScrollPhysics`: Physics for main content scrolling
- `animationStyle`: Custom animation styles
- `resizeToAvoidBottomInset`: Resize when keyboard appears
- `useSafeArea`: Apply safe area constraints
- `modalTypeBuilder`: Default modal type builder

### Custom Animations

Customize pagination and scrolling animations:

```dart
WoltModalSheetThemeData(
  animationStyle: WoltModalSheetAnimationStyle(
    paginationAnimationStyle: WoltModalSheetPaginationAnimationStyle(
      paginationDuration: Duration(milliseconds: 250),
      mainContentIncomingOpacityCurve: Interval(
        0.4,
        1.0,
        curve: Curves.linear,
      ),
      modalSheetHeightTransitionCurve: Interval(
        0.0,
        0.8,
        curve: Curves.fastOutSlowIn,
      ),
    ),
    scrollAnimationStyle: WoltModalSheetScrollAnimationStyle(
      heroImageScaleStart: 1.0,
      heroImageScaleEnd: 0.9,
      topBarTitleTranslationYInPixels: 8.0,
    ),
  ),
)
```

---

## State Management

### Using modalDecorator

**Best practice** for state management: Use `modalDecorator` to wrap the entire modal.

#### With Provider

```dart
class MyViewModel extends ChangeNotifier {
  String _selectedOption = '';
  String get selectedOption => _selectedOption;

  void selectOption(String option) {
    _selectedOption = option;
    notifyListeners();
  }
}

void showModalWithState(BuildContext context) {
  final viewModel = MyViewModel();

  WoltModalSheet.show(
    context: context,
    modalDecorator: (child) => ChangeNotifierProvider.value(
      value: viewModel,
      child: child,
    ),
    pageListBuilder: (modalSheetContext) => [
      WoltModalSheetPage(
        child: Consumer<MyViewModel>(
          builder: (context, viewModel, child) {
            return Column(
              children: [
                Text('Selected: ${viewModel.selectedOption}'),
                ElevatedButton(
                  onPressed: () => viewModel.selectOption('Option 1'),
                  child: Text('Select Option 1'),
                ),
              ],
            );
          },
        ),
      ),
    ],
  );
}
```

#### With Bloc

```dart
void showModalWithBloc(BuildContext context) {
  WoltModalSheet.show(
    context: context,
    modalDecorator: (child) => BlocProvider(
      create: (_) => MyBloc(),
      child: child,
    ),
    pageListBuilder: (modalSheetContext) => [
      WoltModalSheetPage(
        child: BlocBuilder<MyBloc, MyState>(
          builder: (context, state) {
            return YourWidget(state: state);
          },
        ),
      ),
    ],
  );
}
```

### Using pageContentDecorator

For decorating just the page content (not the barrier):

```dart
WoltModalSheet.show(
  context: context,
  pageContentDecorator: (child) => BackdropFilter(
    filter: ImageFilter.blur(sigmaX: 10, sigmaY: 10),
    child: child,
  ),
  pageListBuilder: (context) => [...],
);
```

---

## Common Patterns

### Pattern 1: Conditional Flow

Build dynamic page flows based on user input:

```dart
void showDynamicFlow(BuildContext context) {
  bool needsExtraStep = false;

  WoltModalSheet.show(
    context: context,
    pageListBuilder: (modalSheetContext) {
      final pages = <SliverWoltModalSheetPage>[
        WoltModalSheetPage(
          pageTitle: Text('Step 1'),
          child: Column(
            children: [
              CheckboxListTile(
                title: Text('I need extra configuration'),
                value: needsExtraStep,
                onChanged: (value) {
                  needsExtraStep = value ?? false;
                  if (needsExtraStep) {
                    WoltModalSheet.of(modalSheetContext).addOrReplacePages([
                      _buildExtraConfigPage(),
                      _buildFinalPage(),
                    ]);
                  }
                },
              ),
            ],
          ),
          stickyActionBar: ElevatedButton(
            onPressed: () => WoltModalSheet.of(modalSheetContext).showNext(),
            child: Text('Continue'),
          ),
        ),
      ];

      if (needsExtraStep) {
        pages.add(_buildExtraConfigPage());
      }

      pages.add(_buildFinalPage());

      return pages;
    },
  );
}
```

### Pattern 2: Form Validation Flow

Progress through pages only when valid:

```dart
class FormFlowModal extends StatefulWidget {
  @override
  _FormFlowModalState createState() => _FormFlowModalState();
}

class _FormFlowModalState extends State<FormFlowModal> {
  final _formKey1 = GlobalKey<FormState>();
  final _formKey2 = GlobalKey<FormState>();
  String? email;
  String? password;

  void _showModal() {
    WoltModalSheet.show(
      context: context,
      pageListBuilder: (modalSheetContext) => [
        WoltModalSheetPage(
          pageTitle: Text('Enter Email'),
          child: Form(
            key: _formKey1,
            child: TextFormField(
              decoration: InputDecoration(labelText: 'Email'),
              validator: (value) {
                if (value == null || !value.contains('@')) {
                  return 'Please enter a valid email';
                }
                return null;
              },
              onSaved: (value) => email = value,
            ),
          ),
          stickyActionBar: ElevatedButton(
            onPressed: () {
              if (_formKey1.currentState!.validate()) {
                _formKey1.currentState!.save();
                WoltModalSheet.of(modalSheetContext).showNext();
              }
            },
            child: Text('Continue'),
          ),
        ),
        WoltModalSheetPage(
          pageTitle: Text('Enter Password'),
          leadingNavBarWidget: IconButton(
            icon: Icon(Icons.arrow_back),
            onPressed: () => WoltModalSheet.of(modalSheetContext).showPrevious(),
          ),
          child: Form(
            key: _formKey2,
            child: TextFormField(
              decoration: InputDecoration(labelText: 'Password'),
              obscureText: true,
              validator: (value) {
                if (value == null || value.length < 8) {
                  return 'Password must be at least 8 characters';
                }
                return null;
              },
              onSaved: (value) => password = value,
            ),
          ),
          stickyActionBar: ElevatedButton(
            onPressed: () {
              if (_formKey2.currentState!.validate()) {
                _formKey2.currentState!.save();
                // Submit form
                submitForm(email!, password!);
                Navigator.of(modalSheetContext).pop();
              }
            },
            child: Text('Submit'),
          ),
        ),
      ],
    );
  }

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: _showModal,
      child: Text('Show Form'),
    );
  }
}
```

### Pattern 3: Search with Results

Dynamic content based on search:

```dart
class SearchModal extends StatefulWidget {
  @override
  _SearchModalState createState() => _SearchModalState();
}

class _SearchModalState extends State<SearchModal> {
  List<String> searchResults = [];
  String searchQuery = '';

  void _performSearch(String query) {
    setState(() {
      searchQuery = query;
      // Simulate search
      searchResults = List.generate(
        20,
        (index) => 'Result $index for "$query"',
      );
    });
  }

  void _showModal() {
    WoltModalSheet.show(
      context: context,
      pageListBuilder: (modalSheetContext) => [
        SliverWoltModalSheetPage(
          pageTitle: Text('Search'),
          topBarTitle: Text('Search'),
          trailingNavBarWidget: IconButton(
            icon: Icon(Icons.close),
            onPressed: () => Navigator.of(modalSheetContext).pop(),
          ),
          mainContentSliversBuilder: (context) => [
            SliverToBoxAdapter(
              child: Padding(
                padding: EdgeInsets.all(16),
                child: TextField(
                  decoration: InputDecoration(
                    labelText: 'Search',
                    suffixIcon: Icon(Icons.search),
                  ),
                  onSubmitted: (value) {
                    _performSearch(value);
                    // Rebuild modal to show results
                    WoltModalSheet.of(modalSheetContext).updateCurrentPage(
                      (page) => page,
                    );
                  },
                ),
              ),
            ),
            if (searchResults.isNotEmpty)
              SliverList.builder(
                itemCount: searchResults.length,
                itemBuilder: (context, index) => ListTile(
                  title: Text(searchResults[index]),
                  onTap: () {
                    // Handle selection
                    Navigator.of(modalSheetContext).pop(searchResults[index]);
                  },
                ),
              ),
          ],
        ),
      ],
    );
  }

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      onPressed: _showModal,
      child: Text('Search'),
    );
  }
}
```

### Pattern 4: Loading States

Show loading, error, and success states:

```dart
void showModalWithLoading(BuildContext context) async {
  WoltModalSheet.show(
    context: context,
    pageListBuilder: (modalSheetContext) => [
      // Initial page
      WoltModalSheetPage(
        pageTitle: Text('Submit Data'),
        child: SubmitForm(),
        stickyActionBar: ElevatedButton(
          onPressed: () async {
            // Show loading page
            WoltModalSheet.of(modalSheetContext).pushPage(
              _buildLoadingPage(),
            );

            try {
              // Perform async operation
              await submitDataToServer();

              // Show success page
              WoltModalSheet.of(modalSheetContext).replaceCurrentPage(
                _buildSuccessPage(),
              );
            } catch (e) {
              // Show error page
              WoltModalSheet.of(modalSheetContext).replaceCurrentPage(
                _buildErrorPage(e.toString()),
              );
            }
          },
          child: Text('Submit'),
        ),
      ),
    ],
  );
}

SliverWoltModalSheetPage _buildLoadingPage() {
  return WoltModalSheetPage(
    pageTitle: Text('Submitting...'),
    child: Center(
      child: CircularProgressIndicator(),
    ),
    enableDrag: false, // Prevent dismissal during loading
  );
}

SliverWoltModalSheetPage _buildSuccessPage() {
  return WoltModalSheetPage(
    pageTitle: Text('Success!'),
    child: Column(
      children: [
        Icon(Icons.check_circle, size: 64, color: Colors.green),
        SizedBox(height: 16),
        Text('Your data has been submitted successfully.'),
      ],
    ),
    stickyActionBar: ElevatedButton(
      onPressed: () => Navigator.of(context).pop(),
      child: Text('Close'),
    ),
  );
}

SliverWoltModalSheetPage _buildErrorPage(String error) {
  return WoltModalSheetPage(
    pageTitle: Text('Error'),
    child: Column(
      children: [
        Icon(Icons.error, size: 64, color: Colors.red),
        SizedBox(height: 16),
        Text('An error occurred: $error'),
      ],
    ),
    stickyActionBar: Row(
      children: [
        Expanded(
          child: OutlinedButton(
            onPressed: () => Navigator.of(context).pop(),
            child: Text('Cancel'),
          ),
        ),
        SizedBox(width: 16),
        Expanded(
          child: ElevatedButton(
            onPressed: () => WoltModalSheet.of(context).showAtIndex(0),
            child: Text('Retry'),
          ),
        ),
      ],
    ),
  );
}
```

---

## API Reference

### WoltModalSheet Static Methods

```dart
// Show modal with static page list
static Future<T?> show<T>({
  required BuildContext context,
  required WoltModalSheetPageListBuilder pageListBuilder,
  WoltModalTypeBuilder? modalTypeBuilder,
  ValueNotifier<int>? pageIndexNotifier,
  VoidCallback? onModalDismissedWithBarrierTap,
  VoidCallback? onModalDismissedWithDrag,
  Widget Function(Widget)? pageContentDecorator,
  Widget Function(Widget)? modalDecorator,
  bool? enableDrag,
  bool? showDragHandle,
  bool barrierDismissible = true,
  bool useSafeArea = true,
  RouteSettings? routeSettings,
})

// Show modal with dynamic page list
static Future<T?> showWithDynamicPath<T>({
  required BuildContext context,
  required ValueNotifier<WoltModalSheetPageListBuilder> pageListBuilderNotifier,
  required ValueNotifier<int> pageIndexNotifier,
  WoltModalTypeBuilder? modalTypeBuilder,
  VoidCallback? onModalDismissedWithBarrierTap,
  VoidCallback? onModalDismissedWithDrag,
  Widget Function(Widget)? pageContentDecorator,
  Widget Function(Widget)? modalDecorator,
  bool? enableDrag,
  bool? showDragHandle,
  bool barrierDismissible = true,
  bool useSafeArea = true,
  RouteSettings? routeSettings,
})

// Get WoltModalSheet from context
static WoltModalSheetState of(BuildContext context)
```

### WoltModalSheetState Methods

```dart
// Navigation
bool showNext()
bool showPrevious()
bool showAtIndex(int index)
bool showPageWithId(Object id)
bool showPage<T extends SliverWoltModalSheetPage>({bool Function(T)? where})

// Stack manipulation
void addPage(SliverWoltModalSheetPage page)
void addPages(List<SliverWoltModalSheetPage> pages)
void pushPage(SliverWoltModalSheetPage page)
void pushPages(List<SliverWoltModalSheetPage> pages)
bool popPage()
void replacePage(Object pageId, SliverWoltModalSheetPage newPage)
void replaceCurrentPage(SliverWoltModalSheetPage newPage)
void updateCurrentPage(SliverWoltModalSheetPage Function(SliverWoltModalSheetPage) updateFunction)
void removePage(Object pageId)
void addOrReplacePage(SliverWoltModalSheetPage page)
void addOrReplacePages(List<SliverWoltModalSheetPage> pages)

// Getters
int get currentPageIndex
SliverWoltModalSheetPage get currentPage
List<SliverWoltModalSheetPage> get pages
```

### SliverWoltModalSheetPage Constructor

```dart
SliverWoltModalSheetPage({
  Object? id,
  Widget? pageTitle,
  Widget? topBarTitle,
  Widget? topBar,
  Widget? heroImage,
  required List<Widget> Function(BuildContext) mainContentSliversBuilder,
  Widget? leadingNavBarWidget,
  Widget? trailingNavBarWidget,
  Widget? stickyActionBar,
  bool isTopBarLayerAlwaysVisible = false,
  bool hasTopBarLayer = true,
  bool hasSabGradient = true,
  bool enableDrag = true,
  double navBarHeight = 56.0,
  bool forceMaxHeight = false,
  bool resizeToAvoidBottomInset = true,
})
```

### WoltModalSheetPage Constructor

```dart
WoltModalSheetPage({
  Object? id,
  Widget? pageTitle,
  Widget? topBarTitle,
  Widget? topBar,
  Widget? heroImage,
  required Widget child,
  Widget? leadingNavBarWidget,
  Widget? trailingNavBarWidget,
  Widget? stickyActionBar,
  bool isTopBarLayerAlwaysVisible = false,
  bool hasTopBarLayer = true,
  bool hasSabGradient = true,
  bool enableDrag = true,
  double navBarHeight = 56.0,
  bool forceMaxHeight = false,
  bool resizeToAvoidBottomInset = true,
})
```

### NonScrollingWoltModalSheetPage Constructor

```dart
NonScrollingWoltModalSheetPage({
  Object? id,
  Widget? pageTitle,
  Widget? topBarTitle,
  Widget? topBar,
  required Widget child,
  Widget? leadingNavBarWidget,
  Widget? trailingNavBarWidget,
  Widget? stickyActionBar,
  bool isTopBarLayerAlwaysVisible = false,
  bool hasTopBarLayer = true,
  bool hasSabGradient = false,
  bool enableDrag = true,
  double navBarHeight = 56.0,
  bool resizeToAvoidBottomInset = true,
})
```

---

## Troubleshooting

### Issue: "Cannot find WoltModalSheet.of(context)"

**Problem**: Calling `WoltModalSheet.of(context)` with wrong context.

**Solution**: Use the `modalSheetContext` from `pageListBuilder`:

```dart
WoltModalSheet.show(
  context: context,
  pageListBuilder: (modalSheetContext) => [ // Use this context!
    SliverWoltModalSheetPage(
      trailingNavBarWidget: IconButton(
        onPressed: () => WoltModalSheet.of(modalSheetContext).showNext(), // ✅
        icon: Icon(Icons.arrow_forward),
      ),
    ),
  ],
);
```

### Issue: CupertinoApp crashes with material widgets

**Problem**: Using CupertinoApp but modal uses Material widgets internally.

**Solution**: Add Material localizations delegate:

```dart
CupertinoApp(
  localizationsDelegates: [
    DefaultMaterialLocalizations.delegate,
  ],
  // ... rest of app
)
```

### Issue: Modal doesn't respond to keyboard

**Problem**: Modal doesn't resize when keyboard appears.

**Solution**: Ensure `resizeToAvoidBottomInset` is true (default):

```dart
SliverWoltModalSheetPage(
  resizeToAvoidBottomInset: true, // This is default
  mainContentSliversBuilder: (context) => [
    // Your content with TextField
  ],
)
```

### Issue: Can't scroll list inside modal

**Problem**: List not scrolling, or conflicting with drag-to-dismiss.

**Solution**: Modal handles scrolling automatically. Use `SliverWoltModalSheetPage` with sliver widgets:

```dart
SliverWoltModalSheetPage(
  mainContentSliversBuilder: (context) => [
    SliverList.builder( // ✅ Use sliver widgets
      itemCount: 100,
      itemBuilder: (context, index) => ListTile(
        title: Text('Item $index'),
      ),
    ),
  ],
)
```

### Issue: Top bar not becoming sticky

**Problem**: Top bar doesn't appear when scrolling.

**Solution**: Ensure `hasTopBarLayer` is true and you have scrollable content:

```dart
SliverWoltModalSheetPage(
  hasTopBarLayer: true, // Default
  isTopBarLayerAlwaysVisible: false, // Only show on scroll
  topBarTitle: Text('My Title'),
  mainContentSliversBuilder: (context) => [
    // Must have enough content to scroll
    SliverList.builder(itemCount: 50, ...),
  ],
)
```

### Issue: Modal doesn't adapt to screen size

**Problem**: Modal looks same on all screen sizes.

**Solution**: Use `modalTypeBuilder` for responsive behavior:

```dart
WoltModalSheet.show(
  context: context,
  modalTypeBuilder: (context) {
    final width = MediaQuery.sizeOf(context).width;
    if (width < 524) return WoltModalType.bottomSheet();
    return WoltModalType.dialog();
  },
  pageListBuilder: (context) => [...],
);
```

### Issue: State not persisting across pages

**Problem**: State resets when navigating between pages.

**Solution**: Use `modalDecorator` to provide state management:

```dart
WoltModalSheet.show(
  context: context,
  modalDecorator: (child) => ChangeNotifierProvider(
    create: (_) => MyViewModel(),
    child: child,
  ),
  pageListBuilder: (context) => [...],
);
```

### Issue: Barrier tap not dismissing modal

**Problem**: Tapping outside modal doesn't close it.

**Solution**: Check `barrierDismissible` is true:

```dart
WoltModalSheet.show(
  context: context,
  barrierDismissible: true, // Default
  onModalDismissedWithBarrierTap: () {
    print('Modal dismissed via barrier');
  },
  pageListBuilder: (context) => [...],
);
```

### Issue: Performance issues with long lists

**Problem**: Modal is slow with many items.

**Solution**: Use `SliverList.builder` for lazy loading:

```dart
SliverWoltModalSheetPage(
  mainContentSliversBuilder: (context) => [
    SliverList.builder( // Lazy loads items
      itemCount: 10000,
      itemBuilder: (context, index) => ListTile(
        title: Text('Item $index'),
      ),
    ),
  ],
)
```

---

## Design Guidelines & Breakpoints

### Recommended Breakpoints

- **XSmall**: < 524px → Use `WoltModalType.bottomSheet()`
- **Small**: 524px - 768px → Use `WoltModalType.dialog()`
- **Medium**: 768px - 1400px → Use `WoltModalType.sideSheet()` or `dialog()`
- **Large**: ≥ 1400px → Use `WoltModalType.sideSheet()`

### Best Practices

1. **Always provide close button**: Users should have clear way to dismiss
2. **Limit pages**: 3-5 pages max for optimal UX
3. **Use appropriate page type**: Sliver for lists, regular for static content
4. **Responsive design**: Use `modalTypeBuilder` to adapt to screen size
5. **Clear navigation**: Show back button when not on first page
6. **Sticky actions**: Put primary CTA in `stickyActionBar`
7. **Loading states**: Disable drag during loading operations
8. **Validation**: Validate before allowing navigation to next page
9. **State management**: Use `modalDecorator` for shared state
10. **Accessibility**: Provide meaningful labels and semantic widgets

---

## Quick Reference Card

```dart
// Show basic modal
WoltModalSheet.show(
  context: context,
  pageListBuilder: (ctx) => [/* pages */],
);

// Navigate
WoltModalSheet.of(context).showNext();
WoltModalSheet.of(context).showPrevious();
WoltModalSheet.of(context).showAtIndex(2);

// Stack operations
WoltModalSheet.of(context).addPages([page1, page2]);
WoltModalSheet.of(context).pushPage(newPage);
WoltModalSheet.of(context).updateCurrentPage((page) => page.copyWith(...));

// Responsive
modalTypeBuilder: (context) {
  final w = MediaQuery.sizeOf(context).width;
  return w < 524 ? WoltModalType.bottomSheet() : WoltModalType.dialog();
}

// State management
modalDecorator: (child) => ChangeNotifierProvider(...),

// Page types
SliverWoltModalSheetPage(mainContentSliversBuilder: ...) // Lists/grids
WoltModalSheetPage(child: ...) // Simple widgets
NonScrollingWoltModalSheetPage(child: ...) // No scroll, flex layout
```

---

## Additional Resources

- **GitHub**: https://github.com/woltapp/wolt_modal_sheet
- **pub.dev**: https://pub.dev/packages/wolt_modal_sheet
- **Figma Design Specs**: https://www.figma.com/file/jRQUhvi44bkUxRxSWGhSXO
- **Blog Post**: https://careers.wolt.com/en/blog/engineering/an-overview-of-the-multi-page-scrollable-bottom-sheet-ui-design
- **FlutterCon Talk**: https://www.droidcon.com/2023/08/07/the-art-of-responsive-modals-building-a-multi-page-sheet-in-flutter/

---

*Last updated: 2024 - WoltModalSheet v0.11.0*
