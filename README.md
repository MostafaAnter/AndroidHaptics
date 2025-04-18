# Android Haptics: Proof of Concept
## Using Kotlin and Jetpack Compose

### 1. Introduction

Haptics enhance interactions and convey useful information to users through the sense of touch. When implemented properly, haptic feedback complements visual and audio elements to create a more immersive and responsive experience.

### 2. Haptics Classifications

1. **Clear haptics**: Precise, crisp feedback ideal for UI interactions
2. **Rich haptics**: More complex patterns for enhanced experiences (limited device support)
3. **Buzzy haptics**: Older-style vibrations that feel rough and imprecise (avoid when possible)

### 3. Setup Requirements

```kotlin
// Add to your app's build.gradle
dependencies {
    implementation "androidx.compose.ui:ui:version_number"
    implementation "androidx.compose.material:material:version_number"
}
```
#### Android Haptics: Implementation Approaches

| Approach | Permission Required | API Level | Advantages | Disadvantages | Best For |
|----------|---------------------|-----------|------------|--------------|----------|
| **View-based** | None | All | • Wide device compatibility<br>• Many feedback constants<br>• Respects system settings<br>• Platform consistency | • Limited customization<br>• Requires View access | Standard UI feedback in most apps |
| **LocalHapticFeedback** | None | Compose | • Native Compose integration<br>• Clean API<br>• No View reference needed | • Fewer feedback types<br>• Less control<br>• Still uses View underneath | Compose-focused applications |
| **Predefined VibrationEffect** | VIBRATE | 29+ (Android 10+) | • Hardware-optimized<br>• Standard effects set<br>• Better on supported devices | • Limited to newer devices<br>• Inconsistent across manufacturers<br>• Limited effect options | Special interactions on newer devices |
| **Custom VibrationEffect** | VIBRATE | 26+ (Android 8+) | • Full control over pattern<br>• Custom durations<br>• Intensity control | • Often feels harsh/buzzy<br>• Inconsistent across devices<br>• Battery intensive<br>• Doesn't respect settings | Critical alerts and notifications only |


**Note**: The recommended View-based approach doesn't require the VIBRATE permission, which is an advantage.

### 4. Haptic Feedback Constants Reference

| Constant | Description | Recommended Use Case |
|----------|-------------|-------------|
| `CLOCK_TICK` | The user has pressed either an hour or minute tick of a Clock | Selection changes, minor updates |
| `CONFIRM` | A haptic effect to signal the confirmation or successful completion | Action confirmations, form submissions |
| `CONTEXT_CLICK` | The user has performed a context click on an object | Context menus, secondary actions |
| `DRAG_START` | The user has started a drag-and-drop gesture | Beginning of any drag operation |
| `GESTURE_START` | The user has started a gesture (e.g. on the soft keyboard) | Starting complex gestures |
| `GESTURE_END` | The user has finished a gesture | Completing complex gestures |
| `GESTURE_THRESHOLD_ACTIVATE` | Gesture becomes "eligible" at a threshold of movement | Pull-to-refresh activation |
| `GESTURE_THRESHOLD_DEACTIVATE` | Gesture cancellation by moving back past threshold | Pull-to-refresh cancellation |
| `KEYBOARD_PRESS` | The user has pressed a virtual keyboard key | Virtual keyboard interactions |
| `KEYBOARD_RELEASE` | The user has released a virtual keyboard key | Virtual keyboard interactions |
| `KEYBOARD_TAP` | The user has pressed a soft keyboard key | Virtual keyboard interactions |
| `LONG_PRESS` | The user has performed a long press on an object | Long-press activated actions |
| `REJECT` | A haptic effect to signal the rejection or failure | Error states, failed operations |
| `SEGMENT_TICK` | Switching between a series of potential choices | Sliders, steppers |
| `SEGMENT_FREQUENT_TICK` | Switching between many potential choices | Fine sliders, percentage selection |
| `TEXT_HANDLE_MOVE` | Selection/insertion handle move on text field | Text selection operations |
| `TOGGLE_ON` | The user has toggled a switch into the on position | Switch activation |
| `TOGGLE_OFF` | The user has toggled a switch into the off position | Switch deactivation |
| `VIRTUAL_KEY` | The user has pressed on a virtual on-screen key | Button presses |
| `VIRTUAL_KEY_RELEASE` | The user has released a virtual key | Button releases |

### 5. Draggable Elements with Haptic Feedback

#### Basic Draggable with Haptic Feedback
```kotlin
@Composable
fun BasicDraggableWithHaptics() {
    val view = LocalView.current
    var offsetX by remember { mutableStateOf(0f) }
    val draggableState = rememberDraggableState { delta ->
        offsetX += delta
    }
    
    Box(
        modifier = Modifier
            .fillMaxWidth()
            .height(100.dp)
            .background(Color.LightGray)
            .padding(16.dp)
    ) {
        Box(
            modifier = Modifier
                .size(60.dp)
                .offset { IntOffset(offsetX.roundToInt(), 0) }
                .background(MaterialTheme.colors.primary, shape = CircleShape)
                .draggable(
                    orientation = Orientation.Horizontal,
                    state = draggableState,
                    onDragStarted = {
                        // Provide feedback when drag starts
                        view.performHapticFeedback(HapticFeedbackConstants.DRAG_START)
                    },
                    onDragStopped = {
                        // Provide feedback when drag ends
                        view.performHapticFeedback(HapticFeedbackConstants.GESTURE_END)
                    }
                )
        )
    }
}
```

#### Drag with Threshold Haptic Feedback
```kotlin
@Composable
fun ThresholdDraggableExample() {
    val view = LocalView.current
    var offsetX by remember { mutableStateOf(0f) }
    val thresholdCrossed = remember { mutableStateOf(false) }
    val threshold = 200f
    
    val draggableState = rememberDraggableState { delta ->
        val newOffset = offsetX + delta
        offsetX = newOffset
        
        // Check if we're crossing the threshold
        if (!thresholdCrossed.value && newOffset > threshold) {
            thresholdCrossed.value = true
            view.performHapticFeedback(HapticFeedbackConstants.GESTURE_THRESHOLD_ACTIVATE)
        } else if (thresholdCrossed.value && newOffset <= threshold) {
            thresholdCrossed.value = false
            view.performHapticFeedback(HapticFeedbackConstants.GESTURE_THRESHOLD_DEACTIVATE)
        }
    }
    
    Column(modifier = Modifier.padding(16.dp)) {
        Text("Drag past the threshold")
        Box(
            modifier = Modifier
                .fillMaxWidth()
                .height(100.dp)
                .background(Color.LightGray)
                .padding(16.dp)
        ) {
            // Draw threshold indicator
            Box(
                modifier = Modifier
                    .offset { IntOffset(threshold.toInt(), 0) }
                    .width(2.dp)
                    .height(100.dp)
                    .background(Color.Red)
            )
            
            // Draggable element
            Box(
                modifier = Modifier
                    .size(60.dp)
                    .offset { IntOffset(offsetX.roundToInt(), 0) }
                    .background(
                        color = if (thresholdCrossed.value) Color.Green else MaterialTheme.colors.primary,
                        shape = CircleShape
                    )
                    .draggable(
                        orientation = Orientation.Horizontal,
                        state = draggableState,
                        onDragStarted = {
                            view.performHapticFeedback(HapticFeedbackConstants.DRAG_START)
                        },
                        onDragStopped = {
                            view.performHapticFeedback(HapticFeedbackConstants.GESTURE_END)
                        }
                    )
            )
        }
        
        Text(
            text = if (thresholdCrossed.value) "Threshold activated!" else "Below threshold",
            color = if (thresholdCrossed.value) Color.Green else Color.Gray
        )
    }
}
```

#### Swipe-to-Dismiss with Haptic Feedback
```kotlin
@Composable
fun SwipeToDismissWithHaptics() {
    val view = LocalView.current
    var items by remember { mutableStateOf(List(10) { "Item $it" }) }
    
    LazyColumn {
        items(items) { item ->
            val dismissState = rememberDismissState(
                confirmStateChange = { dismissValue ->
                    when (dismissValue) {
                        DismissValue.DismissedToEnd, DismissValue.DismissedToStart -> {
                            view.performHapticFeedback(HapticFeedbackConstants.CONFIRM)
                            items = items - item
                            true
                        }
                        DismissValue.Default -> false
                    }
                }
            )
            
            // Provide haptic feedback during drag thresholds
            LaunchedEffect(dismissState.progress) {
                when {
                    dismissState.progress.from == DismissValue.Default &&
                    dismissState.progress.to != DismissValue.Default &&
                    dismissState.progress.fraction > 0.1f -> {
                        view.performHapticFeedback(HapticFeedbackConstants.GESTURE_START)
                    }
                    dismissState.targetValue != DismissValue.Default &&
                    dismissState.progress.fraction >= 0.5f -> {
                        view.performHapticFeedback(HapticFeedbackConstants.GESTURE_THRESHOLD_ACTIVATE)
                    }
                }
            }
            
            SwipeToDismiss(
                state = dismissState,
                background = { 
                    val direction = dismissState.dismissDirection
                    val color = when (direction) {
                        SwipeToDismissBox.StartToEnd -> Color.Green
                        SwipeToDismissBox.EndToStart -> Color.Red
                        null -> Color.Transparent
                    }
                    
                    Box(
                        modifier = Modifier
                            .fillMaxSize()
                            .background(color)
                            .padding(16.dp),
                        contentAlignment = when (direction) {
                            SwipeToDismissBox.StartToEnd -> Alignment.CenterStart
                            SwipeToDismissBox.EndToStart -> Alignment.CenterEnd
                            null -> Alignment.Center
                        }
                    ) {
                        Icon(
                            imageVector = when (direction) {
                                SwipeToDismissBox.StartToEnd -> Icons.Default.Check
                                SwipeToDismissBox.EndToStart -> Icons.Default.Delete
                                null -> Icons.Default.KeyboardArrowRight
                            },
                            contentDescription = null,
                            tint = Color.White
                        )
                    }
                },
                dismissContent = {
                    Card(
                        modifier = Modifier
                            .fillMaxWidth()
                            .padding(8.dp),
                        elevation = 4.dp
                    ) {
                        Text(
                            text = item,
                            modifier = Modifier.padding(16.dp)
                        )
                    }
                }
            )
        }
    }
}
```

#### Drag-and-Drop Reorderable List with Haptic Feedback
```kotlin
@Composable
fun ReorderableListWithHaptics() {
    val view = LocalView.current
    var items by remember { mutableStateOf(List(10) { "Item $it" }) }
    val listState = rememberReorderableLazyListState(
        onMove = { from, to ->
            items = items.toMutableList().apply {
                add(to.index, removeAt(from.index))
            }
        },
        onDragStart = {
            view.performHapticFeedback(HapticFeedbackConstants.DRAG_START)
        },
        onDragEnd = {
            view.performHapticFeedback(HapticFeedbackConstants.CONFIRM)
        }
    )
    
    LazyColumn(
        state = listState.listState,
        modifier = Modifier.reorderable(listState)
    ) {
        items(items, { it }) { item ->
            ReorderableItem(listState, key = item) { isDragging ->
                Box(
                    modifier = Modifier
                        .fillMaxWidth()
                        .background(
                            if (isDragging) Color.LightGray else Color.White
                        )
                        .detectReorderAfterLongPress(listState)
                ) {
                    Row(
                        verticalAlignment = Alignment.CenterVertically,
                        modifier = Modifier.padding(16.dp)
                    ) {
                        Icon(Icons.Default.DragHandle, contentDescription = "Drag Handle")
                        Spacer(modifier = Modifier.width(16.dp))
                        Text(item)
                    }
                }
                Divider()
            }
        }
    }
}
```

#### Custom Slider with Segmented Haptic Feedback
```kotlin
@Composable
fun SegmentedHapticSlider() {
    val view = LocalView.current
    var sliderPosition by remember { mutableStateOf(0f) }
    val segments = 10
    var lastSegment by remember { mutableStateOf(0) }
    
    Column(modifier = Modifier.padding(16.dp)) {
        Text("Segmented Slider: ${(sliderPosition * segments).toInt()}")
        
        Slider(
            value = sliderPosition,
            onValueChange = { newPosition ->
                sliderPosition = newPosition
                
                // Calculate which segment we're in
                val currentSegment = (newPosition * segments).toInt()
                
                // If we've moved to a new segment, provide haptic feedback
                if (currentSegment != lastSegment) {
                    // Use SEGMENT_TICK for standard segments
                    // or SEGMENT_FREQUENT_TICK for finer granularity
                    if (segments <= 10) {
                        view.performHapticFeedback(HapticFeedbackConstants.SEGMENT_TICK)
                    } else {
                        view.performHapticFeedback(HapticFeedbackConstants.SEGMENT_FREQUENT_TICK)
                    }
                    lastSegment = currentSegment
                }
            },
            modifier = Modifier.fillMaxWidth()
        )
    }
}
```

#### Pull-to-Refresh with Haptic Thresholds
```kotlin
@Composable
fun PullToRefreshWithHaptics() {
    val view = LocalView.current
    val refreshing = remember { mutableStateOf(false) }
    var thresholdCrossed by remember { mutableStateOf(false) }
    
    val refreshState = rememberPullRefreshState(
        refreshing = refreshing.value,
        onRefresh = {
            refreshing.value = true
            // Simulate a network request
            CoroutineScope(Dispatchers.Main).launch {
                delay(2000)
                refreshing.value = false
            }
        }
    )
    
    // Monitor pull progress for threshold crossing
    LaunchedEffect(refreshState) {
        snapshotFlow { refreshState.progress }
            .collect { progress ->
                if (!refreshing.value) {
                    if (progress >= 1.0f && !thresholdCrossed) {
                        // We just crossed the threshold to activate refresh
                        view.performHapticFeedback(HapticFeedbackConstants.GESTURE_THRESHOLD_ACTIVATE)
                        thresholdCrossed = true
                    } else if (progress < 1.0f && thresholdCrossed) {
                        // We moved back below the threshold
                        view.performHapticFeedback(HapticFeedbackConstants.GESTURE_THRESHOLD_DEACTIVATE)
                        thresholdCrossed = false
                    }
                }
            }
    }
    
    Box(
        modifier = Modifier.fillMaxSize()
    ) {
        PullRefreshIndicator(
            refreshing = refreshing.value,
            state = refreshState,
            modifier = Modifier.align(Alignment.TopCenter)
        )
        
        LazyColumn(
            modifier = Modifier
                .fillMaxSize()
                .pullRefresh(refreshState)
        ) {
            items(20) { index ->
                ListItem(
                    headlineContent = { Text("Item $index") },
                    modifier = Modifier.fillMaxWidth()
                )
                Divider()
            }
        }
    }
}
```

### 6. Recommended Implementation: Using a View in Compose

```kotlin
import androidx.compose.runtime.Composable
import androidx.compose.ui.platform.LocalView
import android.view.HapticFeedbackConstants

@Composable
fun HapticButton(text: String, feedbackType: Int = HapticFeedbackConstants.CONFIRM, onClick: () -> Unit) {
    val view = LocalView.current
    
    Button(
        onClick = { 
            // Trigger haptic feedback
            view.performHapticFeedback(
                feedbackType,
                HapticFeedbackConstants.FLAG_IGNORE_GLOBAL_SETTING
            )
            
            // Perform the actual action
            onClick()
        }
    ) {
        Text(text)
    }
}
```

### 7. Context-Aware Haptic Feedback

```kotlin
@Composable
fun ContextAwareHaptics() {
    val view = LocalView.current
    var isChecked by remember { mutableStateOf(false) }
    
    Switch(
        checked = isChecked,
        onCheckedChange = {
            isChecked = it
            
            // Different haptic feedback based on state
            if (isChecked) {
                view.performHapticFeedback(HapticFeedbackConstants.TOGGLE_ON)
            } else {
                view.performHapticFeedback(HapticFeedbackConstants.TOGGLE_OFF)
            }
        }
    )
}
```

### 8. Best Practices

1. **Use View-based feedback**: This is the recommended approach for most use cases.
2. **Be subtle**: Haptic feedback should enhance, not distract from the user experience.
3. **Be consistent**: Use similar feedback for similar actions throughout your app.
4. **Respect system settings**: Some users may have haptics disabled.
5. **Use appropriate constants**: Match the haptic feedback type to the action being performed.
6. **Test on real devices**: Haptic feedback varies significantly across devices.
7. **Provide fallbacks**: Not all devices support rich haptics.
8. **Don't overuse**: Too much haptic feedback can be annoying or drain battery.

### 9. CAUTION: What to Avoid

⚠️ **IMPORTANT**:
Avoid using older Vibrator methods like `createOneshot` or `createWaveform`, even if they appear to be supported on the device. These modes are often too loud for regular haptic feedback and should only be used for extremely important notifications.

```kotlin
// ❌ AVOID THIS for regular UI feedback
private fun doNotUseForUIFeedback(context: Context) {
    val vibrator = context.getSystemService(Context.VIBRATOR_SERVICE) as Vibrator
    
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
        val effect = VibrationEffect.createOneShot(
            400, // Too long for UI feedback
            VibrationEffect.DEFAULT_AMPLITUDE
        )
        vibrator.vibrate(effect)
    }
}
```

### 10. Main Screen Implementation Example

```kotlin
@Composable
fun HapticFeedbackDemoScreen() {
    var currentTab by remember { mutableStateOf(0) }
    val view = LocalView.current
    val tabs = listOf("Buttons", "Drag", "Lists", "Sliders")
    
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("Android Haptics Demo") }
            )
        },
        bottomBar = {
            TabRow(selectedTabIndex = currentTab) {
                tabs.forEachIndexed { index, title ->
                    Tab(
                        selected = currentTab == index,
                        onClick = {
                            currentTab = index
                            view.performHapticFeedback(HapticFeedbackConstants.CLOCK_TICK)
                        },
                        text = { Text(title) }
                    )
                }
            }
        }
    ) { padding ->
        Box(modifier = Modifier.padding(padding)) {
            when (currentTab) {
                0 -> Column {
                        HapticButton("Confirm Action", HapticFeedbackConstants.CONFIRM) {}
                        HapticButton("Reject Action", HapticFeedbackConstants.REJECT) {}
                     }
                1 -> Column {
                        BasicDraggableWithHaptics()
                        ThresholdDraggableExample()
                     }
                2 -> SwipeToDismissWithHaptics()
                3 -> SegmentedHapticSlider()
            }
        }
    }
}
```

### 11. Conclusion

This POC demonstrates the recommended approach for implementing haptic feedback in Jetpack Compose applications using Kotlin, with special attention to draggable elements and appropriate use of haptic constants. By using the View-based method with the right constants for each interaction type, you can provide clear, meaningful haptic feedback that enhances the user experience.

Remember to use haptics purposefully to:
- Notify users of events that need their attention
- Confirm state changes following user actions
- Enhance gesture interactions with appropriate feedback
- Create intuitive, responsive draggable elements

Always test your haptic implementation on multiple devices to ensure a consistent experience for all users.


### References
- [Implement haptics on Android](https://developer.android.com/develop/ui/views/haptics)
