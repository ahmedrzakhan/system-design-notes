# React Native Interview Questions & Answers

## 1. What is React Native and how is it different from React?

**Q: Explain React Native**

A: React Native is framework for building native mobile apps using JavaScript and React. Write once, run on iOS and Android.

**React vs React Native**:

| React                      | React Native                             |
| -------------------------- | ---------------------------------------- |
| Web UI library             | Mobile app framework                     |
| DOM elements               | Native components                        |
| `<div>`, `<p>`, `<button>` | `<View>`, `<Text>`, `<TouchableOpacity>` |
| Browser renders            | Native OS renders                        |
| CSS styling                | StyleSheet or inline                     |

```jsx
// React Web
<div style={{ color: 'red' }}>
  <button onClick={handleClick}>Click me</button>
</div>

// React Native
<View style={{ color: 'red' }}>
  <TouchableOpacity onPress={handlePress}>
    <Text>Click me</Text>
  </TouchableOpacity>
</View>
```

**Architecture**:

- JavaScript runs in separate thread
- Bridge communicates with native layer
- Native code handles rendering

---

## 2. What are the basic React Native components?

**Q: Core React Native components**

A: React Native has components that map to native ones.

**View** — basic container (like div):

```jsx
<View style={{ flex: 1, justifyContent: "center" }}>
  <Text>Hello</Text>
</View>
```

**Text** — for displaying text (can't use plain strings):

```jsx
<Text style={{ fontSize: 18 }}>Hello, World!</Text>
```

**ScrollView** — scrollable container:

```jsx
<ScrollView>
  <Text>Item 1</Text>
  <Text>Item 2</Text>
  <Text>Item 3</Text>
</ScrollView>
```

**FlatList** — efficient list (for large lists):

```jsx
<FlatList
  data={items}
  renderItem={({ item }) => <Text>{item.name}</Text>}
  keyExtractor={(item) => item.id.toString()}
/>
```

**TextInput** — text input field:

```jsx
<TextInput
  placeholder="Enter name"
  value={name}
  onChangeText={setName}
  style={{ height: 40, borderColor: "gray", borderWidth: 1 }}
/>
```

**TouchableOpacity** — pressable view (with feedback):

```jsx
<TouchableOpacity onPress={handlePress} activeOpacity={0.7}>
  <Text>Press me</Text>
</TouchableOpacity>
```

**TouchableHighlight** — highlight feedback:

```jsx
<TouchableHighlight onPress={handlePress} underlayColor="#DDDDDD">
  <Text>Press me</Text>
</TouchableHighlight>
```

**Button** — simple button:

```jsx
<Button title="Press me" onPress={handlePress} />
```

**Image** — display image:

```jsx
<Image
  source={{ uri: "https://example.com/image.png" }}
  style={{ width: 200, height: 200 }}
/>
```

**Modal** — modal dialog:

```jsx
<Modal visible={showModal} animationType="slide">
  <View>
    <Button title="Close" onPress={() => setShowModal(false)} />
  </View>
</Modal>
```

---

## 3. What is StyleSheet in React Native?

**Q: How do you style in React Native?**

A: StyleSheet optimizes styles by compiling them once. Recommended over inline styles.

```jsx
import { StyleSheet } from "react-native";

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: "#fff",
    padding: 16,
  },
  title: {
    fontSize: 24,
    fontWeight: "bold",
    color: "#000",
  },
  button: {
    backgroundColor: "#007AFF",
    padding: 10,
    borderRadius: 5,
  },
});

function App() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Hello</Text>
    </View>
  );
}
```

**Benefits**:

- Better performance (styles compiled once)
- Error checking (invalid styles caught)
- Clear separation

**Flexbox layout**:

```jsx
const styles = StyleSheet.create({
  container: {
    flex: 1,
    flexDirection: "row", // row or column
    justifyContent: "center", // main axis
    alignItems: "center", // cross axis
  },
  item: {
    flex: 1,
    margin: 10,
  },
});
```

---

## 4. What is FlatList and why use it instead of ScrollView?

**Q: FlatList vs ScrollView**

A: FlatList is optimized for lists. ScrollView renders all items at once.

**ScrollView** — renders all items:

```jsx
<ScrollView>
  {items.map((item) => (
    <Text key={item.id}>{item.name}</Text>
  ))}
</ScrollView>
```

Bad for large lists — all items in memory.

**FlatList** — renders visible items only (virtualization):

```jsx
<FlatList
  data={items}
  renderItem={({ item }) => <Text>{item.name}</Text>}
  keyExtractor={(item) => item.id.toString()}
/>
```

Good for large lists — only visible items rendered.

**Features**:

- Virtualization (only render visible)
- Pull-to-refresh
- Load more on end reached
- Horizontal scrolling

```jsx
<FlatList
  data={items}
  renderItem={({ item }) => <ListItem item={item} />}
  keyExtractor={(item) => item.id.toString()}
  onEndReached={loadMore} // Load more when reach end
  onEndReachedThreshold={0.1}
  refreshing={loading}
  onRefresh={refresh}
  horizontal={true} // Horizontal scroll
/>
```

---

## 5. What is navigation in React Native?

**Q: How do you navigate in React Native?**

A: React Navigation is standard library for navigation.

**Stack navigation** — stack of screens:

```jsx
import { NavigationContainer } from "@react-navigation/native";
import { createNativeStackNavigator } from "@react-navigation/native-stack";

const Stack = createNativeStackNavigator();

function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Details" component={DetailsScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

function DetailsScreen({ navigation, route }) {
  const { params } = route;

  return (
    <View>
      <Text>{params?.itemId}</Text>
      <Button title="Go Back" onPress={() => navigation.goBack()} />
    </View>
  );
}
```

Navigate to screen:

```jsx
navigation.navigate("Details", { itemId: 42 });
```

**Tab navigation** — tabs at bottom:

```jsx
const Tab = createBottomTabNavigator();

<Tab.Navigator>
  <Tab.Screen name="Home" component={HomeScreen} />
  <Tab.Screen name="Settings" component={SettingsScreen} />
</Tab.Navigator>;
```

**Drawer navigation** — slide-out menu:

```jsx
const Drawer = createDrawerNavigator();

<Drawer.Navigator>
  <Drawer.Screen name="Home" component={HomeScreen} />
  <Drawer.Screen name="Settings" component={SettingsScreen} />
</Drawer.Navigator>;
```

---

## 6. What is bridge in React Native?

**Q: Explain React Native bridge**

A: Bridge is communication channel between JavaScript and native code.

**How it works**:

1. JavaScript runs in separate thread
2. Bridge serializes messages
3. Native layer receives and executes
4. Results sent back through bridge

```
JavaScript Thread <-> Bridge <-> Native Thread
```

**Sending message to native**:

```jsx
import { NativeModules } from "react-native";

const { MyNativeModule } = NativeModules;

MyNativeModule.doSomething("hello", (error, result) => {
  if (error) console.error(error);
  else console.log(result);
});
```

**Performance implication**:

- Bridge is bottleneck
- Minimize messages across bridge
- Batch operations
- Avoid too many setState calls from native

**Example — accessing device features**:

```jsx
import { CameraRoll } from "@react-native-community/cameraroll";

// Bridge handles camera roll access
CameraRoll.getPhotos({ first: 20 })
  .then((result) => console.log(result))
  .catch((error) => console.error(error));
```

---

## 7. What are Native Modules and Native Events?

**Q: Explain native modules**

A: Native modules expose native functionality to JavaScript.

**Native module** (iOS Swift):

```swift
@objc(MyModule)
class MyModule: NSObject {
  @objc
  func doSomething(_ message: String, resolver: @escaping RCTPromiseResolveBlock, rejecter: @escaping RCTPromiseRejectBlock) {
    resolver(message.uppercased())
  }
}
```

**Use from JavaScript**:

```jsx
import { NativeModules } from "react-native";

const { MyModule } = NativeModules;

MyModule.doSomething("hello")
  .then((result) => console.log(result)) // "HELLO"
  .catch((error) => console.error(error));
```

**Native events** — listening to native events:

```jsx
import { NativeEventEmitter, NativeModules } from "react-native";

const MyModule = NativeModules.MyModule;
const eventEmitter = new NativeEventEmitter(MyModule);

const subscription = eventEmitter.addListener("eventName", (event) =>
  console.log(event)
);

// Cleanup
subscription.remove();
```

---

## 8. What are hooks in React Native?

**Q: Do hooks work the same in React Native?**

A: Yes, hooks work the same as React. useState, useEffect, etc.

```jsx
function App() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    console.log("Component mounted");
  }, []);

  return (
    <View>
      <Text>{count}</Text>
      <TouchableOpacity onPress={() => setCount(count + 1)}>
        <Text>Increment</Text>
      </TouchableOpacity>
    </View>
  );
}
```

All React hooks work in React Native:

- useState
- useEffect
- useContext
- useReducer
- useCallback
- useMemo
- useRef
- Custom hooks

---

## 9. What is AsyncStorage?

**Q: How do you store local data?**

A: AsyncStorage is key-value storage for React Native.

```jsx
import AsyncStorage from "@react-native-async-storage/async-storage";

// Save data
const saveData = async () => {
  try {
    await AsyncStorage.setItem("username", "john");
  } catch (error) {
    console.error(error);
  }
};

// Retrieve data
const getData = async () => {
  try {
    const value = await AsyncStorage.getItem("username");
    console.log(value); // 'john'
  } catch (error) {
    console.error(error);
  }
};

// Remove data
const removeData = async () => {
  try {
    await AsyncStorage.removeItem("username");
  } catch (error) {
    console.error(error);
  }
};

// Clear all
await AsyncStorage.clear();
```

**Limitations**:

- Simple key-value store only
- Limited size (depends on device)
- Not encrypted

**Better alternatives**:

- WatermelonDB (SQL-like)
- Realm (object database)
- SQLite

---

## 10. What is Permissions in React Native?

**Q: How do you handle permissions?**

A: React Native Permission library for accessing device features.

```jsx
import { PERMISSIONS, RESULTS, request } from "react-native-permissions";

const requestCameraPermission = async () => {
  const result = await request(
    Platform.OS === "ios" ? PERMISSIONS.IOS.CAMERA : PERMISSIONS.ANDROID.CAMERA
  );

  if (result === RESULTS.GRANTED) {
    console.log("Permission granted");
  } else if (result === RESULTS.DENIED) {
    console.log("Permission denied");
  }
};
```

**Common permissions**:

- Camera
- Photo library
- Microphone
- Location
- Contacts
- Calendar

**Android native**:

```jsx
import { PermissionsAndroid } from "react-native";

const requestLocation = async () => {
  const granted = await PermissionsAndroid.request(
    PermissionsAndroid.PERMISSIONS.ACCESS_FINE_LOCATION,
    {
      title: "Location Permission",
      message: "We need access to your location",
    }
  );
};
```

---

## 11. What is Geolocation in React Native?

**Q: How do you get user location?**

A: Use Geolocation API.

```jsx
import Geolocation from "@react-native-community/geolocation";

const getLocation = () => {
  Geolocation.getCurrentPosition(
    (position) => {
      console.log(position.coords.latitude);
      console.log(position.coords.longitude);
    },
    (error) => console.error(error),
    { enableHighAccuracy: true, timeout: 20000 }
  );
};

const watchLocation = () => {
  const watchId = Geolocation.watchPosition(
    (position) => {
      console.log("New location:", position.coords);
    },
    (error) => console.error(error)
  );

  // Stop watching
  Geolocation.clearWatch(watchId);
};
```

---

## 12. What is API calls and networking?

**Q: How do you make API calls in React Native?**

A: Use fetch or Axios (same as web).

**Fetch**:

```jsx
const fetchData = async () => {
  try {
    const response = await fetch("https://api.example.com/data");
    const json = await response.json();
    console.log(json);
  } catch (error) {
    console.error(error);
  }
};
```

**Axios**:

```jsx
import axios from "axios";

const fetchData = async () => {
  try {
    const { data } = await axios.get("https://api.example.com/data");
    console.log(data);
  } catch (error) {
    console.error(error);
  }
};
```

**In useEffect**:

```jsx
useEffect(() => {
  const fetchData = async () => {
    const response = await fetch("https://api.example.com/data");
    const json = await response.json();
    setData(json);
  };

  fetchData();
}, []);
```

**Error handling**:

```jsx
const fetchData = async () => {
  try {
    setLoading(true);
    const response = await fetch(url);
    if (!response.ok) throw new Error("Network error");
    const json = await response.json();
    setData(json);
  } catch (error) {
    setError(error.message);
  } finally {
    setLoading(false);
  }
};
```

---

## 13. What is Platform module?

**Q: How do you write platform-specific code?**

A: Platform module detects OS and run different code.

```jsx
import { Platform, StyleSheet } from "react-native";

// Platform-specific styles
const styles = StyleSheet.create({
  container: {
    padding: Platform.OS === "ios" ? 20 : 10,
  },
});

// Platform-specific components
const getComponent = () => {
  if (Platform.OS === "ios") {
    return require("./ComponentIOS").default;
  } else {
    return require("./ComponentAndroid").default;
  }
};

// Platform.select()
const Component = Platform.select({
  ios: () => require("./ComponentIOS").default,
  android: () => require("./ComponentAndroid").default,
  default: () => require("./ComponentWeb").default,
})();

// Conditional rendering
{
  Platform.OS === "android" && <StatusBar barStyle="dark-content" />;
}
```

**Platform.Version**:

```jsx
if (Platform.OS === "android" && Platform.Version >= 29) {
  // Android 10+
}
```

---

## 14. What is TouchableOpacity, TouchableHighlight, Pressable?

**Q: Difference between touchable components**

A:

**TouchableOpacity** — decreases opacity on press:

```jsx
<TouchableOpacity onPress={handlePress} activeOpacity={0.7}>
  <Text>Press me</Text>
</TouchableOpacity>
```

**TouchableHighlight** — shows highlight color:

```jsx
<TouchableHighlight onPress={handlePress} underlayColor="#DDDDDD">
  <Text>Press me</Text>
</TouchableHighlight>
```

**Pressable** (newer, more flexible):

```jsx
<Pressable
  onPress={handlePress}
  onLongPress={handleLongPress}
  android_ripple={{ color: "rgba(0, 0, 0, 0.2)" }}
  style={({ pressed }) => (pressed ? { opacity: 0.5 } : { opacity: 1 })}
>
  <Text>Press me</Text>
</Pressable>
```

**Pressable features**:

- LongPress
- Ripple effect (Android)
- Hover effects
- Style based on state
- Better for custom interactions

---

## 15. What is debugging in React Native?

**Q: How do you debug React Native?**

A: Multiple debugging methods.

**Console logs**:

```jsx
console.log("Debug message");
console.warn("Warning");
console.error("Error");
```

**React Native Debugger**:

- Chrome DevTools
- Flipper (Flipper from Meta)
- Built-in remote debugger

**Enable debugging**:

```jsx
// On device: shake phone → Open JS Debugger
// Emulator: Ctrl+M (Android) or Cmd+D (iOS)
```

**Performance monitoring**:

```jsx
import { YellowBox } from "react-native";

// Ignore specific warnings
YellowBox.ignoreWarnings(["Warning: ..."]);

// Performance logger
const logger = require("react-native").PerformanceObserver;
```

**React Native Debugger setup**:

```bash
npm install --save-dev react-native-debugger
```

**Breakpoints** — in DevTools, set breakpoints like web.

---

## 16. What is Redux in React Native?

**Q: How do you use state management?**

A: Redux works the same in React Native.

```jsx
import { createStore } from "redux";
import { Provider, useDispatch, useSelector } from "react-redux";

// Reducer
const reducer = (state = { count: 0 }, action) => {
  switch (action.type) {
    case "INCREMENT":
      return { count: state.count + 1 };
    case "DECREMENT":
      return { count: state.count - 1 };
    default:
      return state;
  }
};

const store = createStore(reducer);

// App
function App() {
  return (
    <Provider store={store}>
      <Counter />
    </Provider>
  );
}

function Counter() {
  const dispatch = useDispatch();
  const count = useSelector((state) => state.count);

  return (
    <View>
      <Text>{count}</Text>
      <TouchableOpacity onPress={() => dispatch({ type: "INCREMENT" })}>
        <Text>+</Text>
      </TouchableOpacity>
    </View>
  );
}
```

**Zustand (modern alternative)**:

```jsx
import create from "zustand";

const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
}));

function Counter() {
  const { count, increment } = useStore();
  return (
    <View>
      <Text>{count}</Text>
      <TouchableOpacity onPress={increment}>
        <Text>+</Text>
      </TouchableOpacity>
    </View>
  );
}
```

---

## 17. What is animation in React Native?

**Q: How do you animate in React Native?**

A: Animated API provides smooth animations.

**Basic animation**:

```jsx
import { Animated, View, Button } from "react-native";
import { useEffect, useRef } from "react";

function AnimatedView() {
  const translateX = useRef(new Animated.Value(0)).current;

  const animate = () => {
    Animated.timing(translateX, {
      toValue: 100,
      duration: 500,
      useNativeDriver: true,
    }).start();
  };

  return (
    <View>
      <Animated.View style={{ transform: [{ translateX }] }}>
        <Text>Animated</Text>
      </Animated.View>
      <Button title="Animate" onPress={animate} />
    </View>
  );
}
```

**Interpolation**:

```jsx
const opacity = translateX.interpolate({
  inputRange: [0, 100],
  outputRange: [1, 0],
});
```

**React Native Reanimated** — more powerful:

```jsx
import Animated, {
  useSharedValue,
  withSpring,
  useAnimatedStyle,
} from "react-native-reanimated";

function Reanimated() {
  const offset = useSharedValue(0);

  const animatedStyle = useAnimatedStyle(() => {
    return {
      transform: [{ translateX: offset.value }],
    };
  });

  const handlePress = () => {
    offset.value = withSpring(100);
  };

  return (
    <Animated.View style={animatedStyle}>
      <Text>Animated</Text>
    </Animated.View>
  );
}
```

---

## 18. What is SafeAreaView?

**Q: What is SafeAreaView?**

A: Component that ensures content doesn't go under notches/safe areas.

```jsx
import { SafeAreaView } from "react-native";

function App() {
  return (
    <SafeAreaView style={{ flex: 1 }}>
      <Text>Safe area content</Text>
    </SafeAreaView>
  );
}
```

Important for:

- iPhone notch
- Status bar
- Navigation bar
- Safe area insets

---

## 19. What is useCallback and useMemo in React Native?

**Q: Performance optimization same as React?**

A: Yes, same hooks work in React Native.

```jsx
import { useCallback, useMemo } from "react";

function Parent() {
  const [count, setCount] = useState(0);

  // Memoize function
  const handlePress = useCallback(() => {
    console.log("Pressed");
  }, []);

  // Memoize value
  const expensiveValue = useMemo(() => {
    return compute(count);
  }, [count]);

  return (
    <View>
      <Child onPress={handlePress} />
    </View>
  );
}
```

---

## 20. What is testing in React Native?

**Q: How do you test React Native?**

A: Use Jest and React Native Testing Library.

```jsx
import { render, fireEvent } from "@testing-library/react-native";
import { Counter } from "./Counter";

describe("Counter", () => {
  it("increments on button press", () => {
    const { getByText } = render(<Counter />);

    const button = getByText("+");
    fireEvent.press(button);

    expect(getByText("1")).toBeTruthy();
  });
});
```

**Setup**:

```bash
npm install --save-dev @testing-library/react-native jest
```

**Jest config**:

```json
{
  "preset": "react-native"
}
```

---

## 21. What is the difference between React and React Native?

**Q: Key differences**

A:

| React           | React Native         |
| --------------- | -------------------- |
| `<div>`         | `<View>`             |
| `<p>`           | `<Text>`             |
| `<button>`      | `<TouchableOpacity>` |
| CSS             | StyleSheet           |
| Browser APIs    | Native APIs          |
| Web deployment  | App store            |
| Developer tools | Different debugging  |
| hot reload      | Fast refresh         |

**Learning curve** — If you know React, React Native is easy.

---

## 22. What is Expo?

**Q: What is Expo and should you use it?**

A: Expo is managed environment for React Native development.

**Advantages**:

- No native code needed
- Easy setup
- Fast development
- OTA updates
- Built-in APIs (camera, location, etc.)

**Disadvantages**:

- Limited native modules
- Can't customize native code
- Larger app size
- Publishing through Expo

**Setup**:

```bash
npm install -g expo-cli
expo init MyApp
expo start
```

**Expo vs bare React Native**:

- Use Expo for prototyping
- Use bare React Native for production with heavy native code

---

## 23. What is AppState?

**Q: How do you track app lifecycle?**

A: AppState monitors app foreground/background state.

```jsx
import { AppState } from "react-native";
import { useEffect, useRef, useState } from "react";

function App() {
  const appState = useRef(AppState.currentState);
  const [appStateVisible, setAppStateVisible] = useState(appState.current);

  useEffect(() => {
    const subscription = AppState.addEventListener(
      "change",
      handleAppStateChange
    );

    return () => subscription.remove();
  }, []);

  const handleAppStateChange = (nextAppState) => {
    if (
      appState.current.match(/inactive|background/) &&
      nextAppState === "active"
    ) {
      console.log("App has come to foreground");
    }

    appState.current = nextAppState;
    setAppStateVisible(appState.current);
  };

  return <Text>App state: {appStateVisible}</Text>;
}
```

---

## 24. What is NetInfo?

**Q: How do you check network connectivity?**

A: NetInfo module checks network state.

```jsx
import NetInfo from "@react-native-community/netinfo";

useEffect(() => {
  const unsubscribe = NetInfo.addEventListener((state) => {
    console.log("Is connected:", state.isConnected);
    console.log("Type:", state.type);
  });

  return () => unsubscribe();
}, []);

// Check once
NetInfo.fetch().then((state) => {
  console.log("Is connected:", state.isConnected);
});
```

---

## 25. What is orientation in React Native?

**Q: How do you handle screen rotation?**

A: Use Dimensions API.

```jsx
import { Dimensions } from "react-native";

function App() {
  const [dimensions, setDimensions] = useState(Dimensions.get("window"));

  useEffect(() => {
    const subscription = Dimensions.addEventListener("change", ({ window }) => {
      setDimensions(window);
    });

    return () => subscription?.remove();
  }, []);

  return (
    <View>
      <Text>Width: {dimensions.width}</Text>
      <Text>Height: {dimensions.height}</Text>
    </View>
  );
}
```

**React Native Orientation Library** (for locking orientation):

```jsx
import Orientation from "react-native-orientation-locker";

Orientation.lockToPortrait(); // Lock to portrait
Orientation.unlockAllOrientations(); // Unlock
```

---

## React Native Interview Tips

1. **React knowledge transfers** — If you know React, you know most of RN
2. **Native bridges** — Understand how JS talks to native code
3. **Performance matters** — Mobile has limited resources
4. **FlatList over ScrollView** — Key optimization
5. **Platform differences** — iOS and Android are different
6. **Testing** — Know Jest and testing library
7. **Production experience** — Talk about real apps
8. **Debugging** — Know the tools (Flipper, DevTools)
9. **Permissions** — Apps need permission for features
10. **Common libraries** — React Navigation, Axios, Redux
