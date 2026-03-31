# /wiom-frontend-dev — Wiom Frontend Dev

Wiom's Kotlin/Compose frontend development standards. Activate when user says "wiom frontend dev", "load wiom frontend", or `/wiom-frontend-dev`.

Generate a complete Jetpack Compose Android screen or full application based on the user's description, following Wiom's engineering standards below.

## How to use
The user will describe the screens they want. Parse their request and generate all required Kotlin files.

## Rules — always follow these

### 1. @Preview annotations (MANDATORY)
Every screen composable MUST have `@Preview` annotations. Never skip this.
- Add **2 previews per screen** when the app is bilingual (one per language)
- Use `showBackground = true` and `showSystemUi = true`
- Preview functions must be `private` and named `<ScreenName><Language>Preview`
- Import `androidx.compose.ui.tooling.preview.Preview`

Example:
```kotlin
@Preview(name = "Login Screen - English", showBackground = true, showSystemUi = true)
@Composable
private fun LoginScreenEnglishPreview() {
    WiomAuthTheme {
        LoginScreen(strings = EnglishStrings, ...)
    }
}

@Preview(name = "Login Screen - Hindi", showBackground = true, showSystemUi = true)
@Composable
private fun LoginScreenHindiPreview() {
    WiomAuthTheme {
        LoginScreen(strings = HindiStrings, ...)
    }
}
```

### 2. Bilingual support (Hindi + English)
- Use a `AppStrings` data class with all UI strings
- Provide `EnglishStrings` and `HindiStrings` instances
- Use `AuthViewModel` with `StateFlow<AppStrings>` for runtime switching (no activity recreation)
- Add a language toggle button (`FilledTonalButton`) on every screen

### 3. Project structure
```
app/src/main/java/com/wiom/<appname>/
├── MainActivity.kt
├── data/AppStrings.kt          ← all bilingual strings
├── viewmodel/AuthViewModel.kt  ← language + auth state
├── navigation/
│   ├── Screen.kt
│   └── NavGraph.kt
└── ui/
    ├── screens/
    │   ├── LoginScreen.kt
    │   ├── ForgotPasswordScreen.kt
    │   └── HomeScreen.kt
    └── theme/
        ├── Color.kt
        ├── Dimens.kt   ← all dp/sp constants
        ├── Theme.kt
        └── Type.kt
```

### 4. Gradle versions (adapt to user's environment)
Do NOT hardcode versions. Instead:
- **Ask the user for their Android Studio version** if not mentioned, or check if they have a existing project with versions already set
- Use the latest AGP version **compatible with their Android Studio** (e.g. Android Studio Hedgehog supports max AGP 8.2, Koala supports AGP 8.5, etc.)
- Match the Gradle wrapper version to what the chosen AGP requires
- Use the latest stable Kotlin, Compose BOM, and library versions compatible with the chosen AGP
- If a build fails due to version mismatch, adjust to fit the user's system — do not force specific versions

Reference defaults (only if user provides no context):
- AGP: latest stable compatible with their Android Studio
- Kotlin: `2.0.0`
- Compose BOM: `2024.10.01`
- Navigation Compose: `2.8.3`
- ViewModel Compose: `2.8.6`
- Activity Compose: `1.9.3`

### 5. Screen standards
- Use **Material 3** components only
- All screens use `Box` with `Brush.verticalGradient` background
- Screens must be scrollable: wrap content in `verticalScroll(rememberScrollState())`
- Use `systemBarsPadding()` for edge-to-edge support

### 5a. Constants file (MANDATORY — no hardcoded values)
Never use raw `dp` or `sp` values directly in composables. Always define them in a constants file and reference from there.

Create `ui/theme/Dimens.kt`:
```kotlin
package com.wiom.<appname>.ui.theme

import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp

object Dimens {
    // Padding & Spacing
    val paddingSmall = 8.dp
    val paddingMedium = 16.dp
    val paddingLarge = 24.dp
    val paddingXLarge = 32.dp

    // Corner radius
    val cornerSmall = 12.dp
    val cornerMedium = 16.dp
    val cornerLarge = 20.dp
    val cornerFull = 50.dp

    // Component sizes
    val buttonHeight = 52.dp
    val avatarSize = 100.dp
    val avatarIconSize = 60.dp
    val iconSize = 44.dp
    val iconSizeSmall = 18.dp
    val stepIndicatorSize = 32.dp
    val cardElevation = 6.dp

    // Font sizes
    val fontSizeSmall = 11.sp
    val fontSizeCaption = 12.sp
    val fontSizeBody = 13.sp
    val fontSizeDefault = 14.sp
    val fontSizeSubtitle = 15.sp
    val fontSizeMedium = 16.sp
    val fontSizeTitle = 18.sp
    val fontSizeHeading = 22.sp
    val fontSizeLarge = 28.sp
    val fontSizeXLarge = 30.sp
    val fontSizeDisplay = 36.sp
}
```

Usage in screens:
```kotlin
// ✅ correct
Text(text = strings.login, fontSize = Dimens.fontSizeHeading)
Button(modifier = Modifier.height(Dimens.buttonHeight))
Card(shape = RoundedCornerShape(Dimens.cornerLarge))

// ❌ wrong
Text(text = strings.login, fontSize = 22.sp)
Button(modifier = Modifier.height(52.dp))
```

Add `Dimens.kt` to the project structure and import it in every screen file.

**When working on an existing project:** If the user asks to add screens to an existing project that already has hardcoded values, also refactor those existing screens to use `Dimens.*`. Never mix hardcoded values and `Dimens.*` in the same project.

### 6. Navigation
- Use Jetpack Navigation Compose
- Store shared state (logged-in email, language) in ViewModel — do NOT pass via nav arguments
- On logout: `popUpTo(0) { inclusive = true }` to clear back stack

### 7. Form validation
Always validate:
- Empty fields → show `strings.emptyField`
- Invalid email format → show `strings.invalidEmail`
- Password min 6 chars → show `strings.passwordTooShort`
- Password mismatch → show `strings.passwordMismatch`

### 8. Forgot Password flow
Two-step flow with animated transition:
- **Step 1**: Email field + Validate button → on success set `emailValidated = true`
- **Step 2**: Appears via `AnimatedVisibility(fadeIn + slideInVertically)` — New Password + Confirm Password + Reset button
- Show step indicators at top (`StepChip` composables)
- Show success card with `CheckCircle` icon after reset

### 9. ViewModel rules (MVVM separation)
- All business logic stays in ViewModel — composables only handle UI events and display state
- Expose state via `StateFlow`, collect in composables using `collectAsState()`
- Never call Android APIs (e.g. `Patterns.EMAIL_ADDRESS`) directly in composables — delegate to ViewModel
- ViewModel must not hold references to `Context`, `Activity`, or any composable

```kotlin
// ✅ correct
val strings by viewModel.currentStrings.collectAsState()
val isHindi by viewModel.isHindi.collectAsState()

// ❌ wrong
var strings by remember { mutableStateOf(EnglishStrings) }
```

### 10. Color palette (Color.kt standards)
Define semantic color names in `Color.kt`, not raw hex values scattered in composables.
All custom colors must be defined here and referenced by name:
```kotlin
// ✅ correct
val BrandPrimary = Color(0xFF6650A4)
val BrandSecondary = Color(0xFF625B71)

// ❌ wrong — inline hex in composable
color = Color(0xFF6650A4)
```
Use `MaterialTheme.colorScheme.*` tokens in composables wherever possible — only fall back to custom colors when Material tokens don't cover the use case.

### 11. Accessibility
- `contentDescription` for icons is optional — add it when it meaningfully helps screen reader users, skip it for decorative icons (`contentDescription = null`)
- All `TextField` components must have a `label` or `placeholder`
- `Button` text must always be visible (never icon-only buttons without a label or contentDescription)

### 12. Keyboard & IME handling
- Use appropriate `KeyboardOptions` per field:
  - Email → `KeyboardType.Email`
  - Password → `KeyboardType.Password`
  - Text → `KeyboardType.Text`
- Chain fields using `ImeAction`:
  - All fields except the last → `ImeAction.Next`
  - Last field → `ImeAction.Done`
- Add to `AndroidManifest.xml` activity tag:
  ```xml
  android:windowSoftInputMode="adjustResize"
  ```
  This prevents the keyboard from covering input fields.

### 13. `@OptIn` annotations
Some Material3 APIs are experimental and require opt-in. Always add at composable level, not file level:
```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun MyScreen(...) { ... }
```
Required for: `TopAppBar`, `ExposedDropdownMenuBox`, `DatePicker`, `TimePicker`, `ModalBottomSheet`.

### 14. Error state management
- Clear a field's error as soon as the user starts typing in that field
- Reset ALL errors when language toggles using `LaunchedEffect(strings)`:
```kotlin
LaunchedEffect(strings) {
    emailError = ""
    passwordError = ""
    loginError = ""
}
```

### 15. RTL & AutoMirrored icons
- Use `Icons.AutoMirrored.Filled.*` for directional icons (back arrow, forward arrow, send, etc.) to correctly mirror in RTL layouts
- Regular icons (lock, email, visibility) do not need AutoMirrored

```kotlin
// ✅ correct for navigation
Icon(Icons.AutoMirrored.Filled.ArrowBack, ...)
Icon(Icons.AutoMirrored.Filled.ExitToApp, ...)

// ❌ wrong
Icon(Icons.Default.ArrowBack, ...)
```

### 16. Missing binary files note
`gradlew` and `gradlew.bat` are binary wrapper scripts — they cannot be generated as text files.
After creating the project, tell the user to either:
- Open the project in **Android Studio** (it auto-generates these), OR
- Run `gradle wrapper` in the project root if Gradle is installed globally

Always generate `gradle/wrapper/gradle-wrapper.properties` as it is a text file and is required.

### 17. proguard-rules.pro
Always create `app/proguard-rules.pro` even if empty. The `app/build.gradle.kts` references it and the build will warn without it:
```
# Add project specific ProGuard rules here.
```

### 18. Corner radius — inline RoundedCornerShape
Use `RoundedCornerShape` directly with the radius value that fits the design. No need for a centralized shape constants file.

```kotlin
// ✅ correct — use the radius that matches the design
Card(shape = RoundedCornerShape(16.dp))
Button(shape = RoundedCornerShape(12.dp))
```

### 19. Sealed UI state — async state pattern (MANDATORY for ViewModels with API calls)
Every ViewModel that makes API calls must model its state with a sealed interface, not ad-hoc boolean flags:

```kotlin
sealed interface ScreenUiState {
    data object Idle : ScreenUiState
    data object Loading : ScreenUiState
    data object Success : ScreenUiState
    data class Error(val message: String) : ScreenUiState
}

// In ViewModel:
var uiState by mutableStateOf<ScreenUiState>(ScreenUiState.Idle)
    private set
```

In the composable, use `when` to render the correct UI for each state:
```kotlin
when (val state = viewModel.uiState) {
    is ScreenUiState.Idle -> { /* default UI */ }
    is ScreenUiState.Loading -> LoadingOverlay()
    is ScreenUiState.Success -> { /* success UI */ }
    is ScreenUiState.Error -> ErrorCard(message = state.message)
}
```

Never use separate `isLoading: Boolean` and `errorMessage: String?` fields — use the sealed interface instead.

### 20. ValidationUtils — Separate regex validation object
Keep all regex validation in a pure utility object separate from the ViewModel. The ViewModel calls utilities; utilities don't know about ViewModel or UI.

```kotlin
object ValidationUtils {
    fun isValidMobile(value: String) = value.matches(Regex("^[6-9]\\d{9}$"))
    fun isValidEmail(value: String) = android.util.Patterns.EMAIL_ADDRESS.matcher(value).matches()
    fun isValidOtp(value: String) = value.matches(Regex("^\\d{4,6}$"))
    fun isValidPincode(value: String) = value.matches(Regex("^\\d{6}$"))
    fun isValidPan(value: String) = value.matches(Regex("^[A-Z]{5}\\d{4}[A-Z]$"))
    fun isValidAadhaar(value: String) = value.matches(Regex("^\\d{12}$"))
    fun isValidIfsc(value: String) = value.matches(Regex("^[A-Z]{4}0[A-Z0-9]{6}$"))
}
```

For bilingual validation error messages, create a separate `AppValidation` file:
```kotlin
object AppValidation {
    fun validatePhone(value: String, strings: AppStrings): String? = when {
        value.isEmpty() -> strings.emptyField
        !ValidationUtils.isValidMobile(value) -> strings.invalidPhone
        else -> null
    }
    fun validateEmail(value: String, strings: AppStrings): String? = when {
        value.isEmpty() -> strings.emptyField
        !ValidationUtils.isValidEmail(value) -> strings.invalidEmail
        else -> null
    }
}
```

### 22. LoadingOverlay + ErrorCard — Standard async/error components
Always add these two reusable components to `CommonComponents.kt`. Never build one-off loading/error UI in individual screens.

```kotlin
@Composable
fun LoadingOverlay(message: String = "") {
    Box(
        modifier = Modifier
            .fillMaxSize()
            .background(Color.Black.copy(alpha = 0.4f)),
        contentAlignment = Alignment.Center
    ) {
        Card(shape = RoundedCornerShape(16.dp)) {
            Column(
                modifier = Modifier.padding(Dimens.paddingXLarge),
                horizontalAlignment = Alignment.CenterHorizontally,
                verticalArrangement = Arrangement.spacedBy(Dimens.paddingMedium)
            ) {
                CircularProgressIndicator(color = MaterialTheme.colorScheme.primary)
                if (message.isNotEmpty()) {
                    Text(message, fontSize = Dimens.fontSizeDefault)
                }
            }
        }
    }
}

enum class ErrorCardType { ERROR, WARNING, INFO }

@Composable
fun ErrorCard(message: String, type: ErrorCardType = ErrorCardType.ERROR) {
    val (bgColor, iconColor) = when (type) {
        ErrorCardType.ERROR -> MaterialTheme.colorScheme.errorContainer to MaterialTheme.colorScheme.error
        ErrorCardType.WARNING -> Color(0xFFFFF3E0) to Color(0xFFE65100)
        ErrorCardType.INFO -> MaterialTheme.colorScheme.secondaryContainer to MaterialTheme.colorScheme.secondary
    }
    Card(
        modifier = Modifier.fillMaxWidth(),
        shape = RoundedCornerShape(12.dp),
        colors = CardDefaults.cardColors(containerColor = bgColor)
    ) {
        Text(
            text = message,
            modifier = Modifier.padding(Dimens.paddingMedium),
            color = iconColor,
            fontSize = Dimens.fontSizeDefault
        )
    }
}
```

### 23. `collectAsStateWithLifecycle` — preferred over `collectAsState()`
Always use `collectAsStateWithLifecycle()` instead of `collectAsState()`. It stops collecting when the screen goes to the background, preventing unnecessary work and memory leaks.

```kotlin
// ✅ correct
implementation("androidx.lifecycle:lifecycle-runtime-compose:<version>")

val uiState by viewModel.uiState.collectAsStateWithLifecycle()
val strings by viewModel.currentStrings.collectAsStateWithLifecycle()

// ❌ wrong
val uiState by viewModel.uiState.collectAsState()
```

Add to `app/build.gradle.kts`:
```kotlin
implementation("androidx.lifecycle:lifecycle-runtime-compose:2.8.6")
```

## Output
Generate ALL files needed (build.gradle.kts, AndroidManifest.xml, all Kotlin source files, strings.xml, themes.xml, gradle-wrapper.properties, proguard-rules.pro). Do not skip any file. Ask for the target directory if not specified. After generation, remind the user to open the project in Android Studio to auto-generate the `gradlew` scripts before building.
