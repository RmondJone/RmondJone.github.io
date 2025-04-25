---
title: "Android Fragment详解"
date: 2025-04-25T16:09:24+08:00
draft: false
categories: ["移动端"]
tags: ["Android"]
---

# Android Fragment 详解

Fragment（片段）是 Android 3.0 (API level 11) 引入的重要组件，用于构建灵活、模块化的用户界面，特别适合平板电脑等大屏幕设备。

## Fragment 基本概念

### 什么是 Fragment

1. **UI 模块**：代表 Activity 中的一部分行为或用户界面
2. **可重用组件**：可以在多个 Activity 中重复使用
3. **独立生命周期**：拥有自己的生命周期，但受宿主 Activity 影响
4. **响应配置变化**：比 Activity 更适合处理屏幕旋转等配置变化

### Fragment 与 Activity 的关系

- Fragment 必须嵌入到 Activity 中
- 一个 Activity 可以包含多个 Fragment
- Fragment 的生命周期受宿主 Activity 影响
- Fragment 可以通过 Activity 实现与其他 Fragment 的通信

## Fragment 生命周期

Fragment 生命周期比 Activity 更复杂，包含以下回调方法：

1. **onAttach()** - Fragment 与 Activity 关联时调用
2. **onCreate()** - Fragment 创建时调用
3. **onCreateView()** - 创建 Fragment 的视图层次结构
4. **onActivityCreated()** - Activity 的 onCreate() 完成后调用
5. **onStart()** - Fragment 可见时调用
6. **onResume()** - Fragment 可交互时调用
7. **onPause()** - Fragment 不再可交互时调用
8. **onStop()** - Fragment 不可见时调用
9. **onDestroyView()** - 移除与 Fragment 关联的视图层次结构
10. **onDestroy()** - Fragment 状态最终清理时调用
11. **onDetach()** - Fragment 与 Activity 解除关联时调用

![Fragment 生命周期图](/images/android_fragment.png)

## 创建 Fragment

### 1. 创建 Fragment 类

```java
public class MyFragment extends Fragment {
    
    public MyFragment() {
        // 必须有一个空构造函数
    }
    
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // 膨胀 Fragment 的布局
        return inflater.inflate(R.layout.fragment_my, container, false);
    }
    
    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        // 初始化视图组件
        Button button = view.findViewById(R.id.button);
        button.setOnClickListener(v -> {
            // 处理点击事件
        });
    }
}
```

### 2. 创建 Fragment 布局文件

`res/layout/fragment_my.xml`:
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    
    <Button
        android:id="@+id/button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Click Me" />
</LinearLayout>
```

## 添加 Fragment 到 Activity

### 方式1: 静态添加（XML布局）

```xml
<!-- activity_main.xml -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    
    <fragment
        android:id="@+id/myFragment"
        android:name="com.example.MyFragment"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</LinearLayout>
```

### 方式2: 动态添加（代码）

```java
// 获取 FragmentManager
FragmentManager fragmentManager = getSupportFragmentManager();
// 开始事务
FragmentTransaction transaction = fragmentManager.beginTransaction();
// 添加 Fragment
transaction.add(R.id.fragment_container, new MyFragment());
// 提交事务
transaction.commit();
```

## Fragment 事务

FragmentTransaction 允许对 Fragment 执行添加、移除、替换等操作：

```java
FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();

// 替换 Fragment
transaction.replace(R.id.fragment_container, new AnotherFragment());

// 添加到返回栈，按返回键可回到前一个 Fragment
transaction.addToBackStack(null);

transaction.commit();
```

## Fragment 间通信

### 1. 通过 Activity 中介

```java
// 在 Fragment 中
((HostActivity) getActivity()).communicateWithOtherFragment(data);

// 在 Activity 中
public void communicateWithOtherFragment(String data) {
    OtherFragment fragment = (OtherFragment) getSupportFragmentManager()
            .findFragmentById(R.id.other_fragment);
    fragment.receiveData(data);
}
```

### 2. 使用接口回调

```java
// 在 Fragment 中定义接口
public interface OnFragmentInteractionListener {
    void onFragmentInteraction(String data);
}

// Activity 实现接口
public class MainActivity extends AppCompatActivity implements MyFragment.OnFragmentInteractionListener {
    @Override
    public void onFragmentInteraction(String data) {
        // 处理来自 Fragment 的数据
    }
}

// 在 Fragment 的 onAttach 中设置监听器
@Override
public void onAttach(Context context) {
    super.onAttach(context);
    if (context instanceof OnFragmentInteractionListener) {
        listener = (OnFragmentInteractionListener) context;
    } else {
        throw new RuntimeException(context.toString()
                + " must implement OnFragmentInteractionListener");
    }
}

// 发送数据
listener.onFragmentInteraction("Hello from Fragment!");
```

### 3. 使用 ViewModel (推荐)

```java
// 共享的 ViewModel
public class SharedViewModel extends ViewModel {
    private final MutableLiveData<String> selected = new MutableLiveData<>();
    
    public void select(String item) {
        selected.setValue(item);
    }
    
    public LiveData<String> getSelected() {
        return selected;
    }
}

// 在 Fragment 中
SharedViewModel model = new ViewModelProvider(requireActivity()).get(SharedViewModel.class);
model.getSelected().observe(getViewLifecycleOwner(), item -> {
    // 更新 UI
});

// 发送数据
model.select("New data");
```

## Fragment 类型

### 1. 普通 Fragment

基本的 UI Fragment，如上例所示

### 2. DialogFragment

显示对话框的专用 Fragment：

```java
public class MyDialogFragment extends DialogFragment {
    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        AlertDialog.Builder builder = new AlertDialog.Builder(getActivity());
        builder.setTitle("提示")
               .setMessage("这是一个对话框")
               .setPositiveButton("确定", (dialog, id) -> {});
        return builder.create();
    }
}

// 显示对话框
MyDialogFragment dialog = new MyDialogFragment();
dialog.show(getSupportFragmentManager(), "MyDialog");
```

### 3. PreferenceFragmentCompat

用于创建设置界面：

```java
public class SettingsFragment extends PreferenceFragmentCompat {
    @Override
    public void onCreatePreferences(Bundle savedInstanceState, String rootKey) {
        setPreferencesFromResource(R.xml.preferences, rootKey);
    }
}
```

## Fragment 与 ViewPager

Fragment 常与 ViewPager 结合实现滑动标签页：

```java
public class SectionsPagerAdapter extends FragmentPagerAdapter {
    public SectionsPagerAdapter(FragmentManager fm) {
        super(fm, BEHAVIOR_RESUME_ONLY_CURRENT_FRAGMENT);
    }
    
    @Override
    public Fragment getItem(int position) {
        // 返回对应位置的 Fragment
        return PlaceholderFragment.newInstance(position + 1);
    }
    
    @Override
    public int getCount() {
        return 3; // 页数
    }
}

// 在 Activity 中设置
ViewPager viewPager = findViewById(R.id.view_pager);
viewPager.setAdapter(new SectionsPagerAdapter(getSupportFragmentManager()));
```

## Fragment 最佳实践

1. **避免在 Fragment 中直接引用其他 Fragment**：使用接口或 ViewModel 通信
2. **正确处理配置变化**：使用 setRetainInstance(true) 保留非 UI 数据（已废弃，推荐使用 ViewModel）
3. **谨慎使用 addToBackStack**：避免创建过深的返回栈
4. **使用 FragmentFactory**：AndroidX 提供 FragmentFactory 来改进 Fragment 实例化
5. **注意内存泄漏**：避免在 Fragment 中持有 Activity 的长生命周期引用
6. **使用 Navigation 组件**：简化 Fragment 导航和管理

## 常见问题解决方案

### 1. Fragment 重叠问题

在 Activity 的 onCreate 中添加 Fragment 时检查 savedInstanceState：

```java
if (savedInstanceState == null) {
    getSupportFragmentManager().beginTransaction()
            .add(R.id.container, new MyFragment())
            .commit();
}
```

### 2. getActivity() 返回 null

确保在 Fragment 生命周期正确阶段调用，或使用 requireActivity()

### 3. 视图查找时机

在 onViewCreated() 中执行视图查找和初始化，而不是 onCreateView()

### 4. 正确处理用户可见性

使用 setUserVisibleHint() (已废弃) 或 ViewPager2 的 FragmentStateAdapter 的 BEHAVIOR_RESUME_ONLY_CURRENT_FRAGMENT

## 现代 Fragment 开发 (AndroidX)

1. **使用 FragmentContainerView** 替代 FrameLayout 作为 Fragment 容器
2. **使用新的 Fragment 结果 API** 替代 startActivityForResult
```java
// 设置结果监听器
setFragmentResultListener("requestKey", this, (key, bundle) -> {
    // 处理结果
});

// 发送结果
Bundle result = new Bundle();
result.putString("data", "some data");
setFragmentResult("requestKey", result);
```
3. **使用 Navigation 组件** 管理 Fragment 导航
4. **使用 ViewModel 和 LiveData** 管理数据

Fragment 是 Android 开发中构建灵活 UI 的强大工具，合理使用可以创建适应不同屏幕尺寸的高质量应用。随着 Android 开发的发展，Fragment 的最佳实践也在不断演进，建议结合最新的 Jetpack 组件使用。
