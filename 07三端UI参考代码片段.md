# 三端 UI 参考代码片段

适用原型：`鸥我白盒8.2-改ui-开发需求备注版-GitHub最终交付版.html` 最终交付版  
适用文档：`UI改版技术交付文档 V1.0.md`  
说明：以下代码只提供关键页面核心布局，不包含完整业务逻辑、网络请求、路由和持久化。尺寸按交付文档 token 映射：iOS 使用 pt，Android 使用 dp/sp，鸿蒙使用 vp/fp。

## 0. 通用主题 Token

### iOS SwiftUI

```swift
import SwiftUI

enum OwoColor {
    static let brand = Color(red: 159/255, green: 151/255, blue: 235/255) // #9F97EB 品牌主色
    static let brandStrong = Color(red: 126/255, green: 114/255, blue: 217/255)
    static let success = Color(red: 5/255, green: 150/255, blue: 105/255)
    static let warning = Color(red: 245/255, green: 158/255, blue: 11/255)
    static let error = Color(red: 239/255, green: 68/255, blue: 68/255)

    static func appBackground(_ scheme: ColorScheme) -> Color {
        scheme == .dark ? Color(red: 18/255, green: 18/255, blue: 18/255) : Color(.systemBackground)
    }

    static func card(_ scheme: ColorScheme) -> Color {
        scheme == .dark ? Color(red: 31/255, green: 31/255, blue: 39/255) : Color(.secondarySystemBackground)
    }

    static func primaryText(_ scheme: ColorScheme) -> Color {
        scheme == .dark ? Color(red: 245/255, green: 245/255, blue: 247/255) : Color(.label)
    }

    static func secondaryText(_ scheme: ColorScheme) -> Color {
        scheme == .dark ? Color(red: 199/255, green: 199/255, blue: 204/255) : Color(.secondaryLabel)
    }
}

enum OwoMetric {
    static let pagePadding: CGFloat = 16
    static let harmonyPagePadding: CGFloat = 24
    static let cardRadius: CGFloat = 16
    static let buttonRadius: CGFloat = 14
    static let sheetRadius: CGFloat = 28
    static let minTouch: CGFloat = 44 // iOS 最小点击区域 44×44pt，符合 HIG
}
```

### Android Jetpack Compose

```kotlin
import android.app.Activity
import android.os.Build
import androidx.compose.foundation.isSystemInDarkTheme
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.platform.LocalContext

private val OwoBrand = Color(0xFF9F97EB)
private val OwoBrandStrong = Color(0xFF7E72D9)
private val OwoSuccess = Color(0xFF059669)
private val OwoWarning = Color(0xFFF59E0B)
private val OwoError = Color(0xFFEF4444)

private val OwoLightColors = lightColorScheme(
    primary = OwoBrand,
    onPrimary = Color.White,
    primaryContainer = Color(0xFFEDE9FE),
    background = Color(0xFFFFFFFF),
    surface = Color(0xFFFFFFFF),
    surfaceVariant = Color(0xFFF8F8FC),
    onSurface = Color(0xFF111111),
    onSurfaceVariant = Color(0xFF555555),
    outlineVariant = Color(0xFFECECEC),
    error = OwoError
)

private val OwoDarkColors = darkColorScheme(
    primary = OwoBrand,
    onPrimary = Color.White,
    primaryContainer = Color(0xFF3B2167),
    background = Color(0xFF121212),
    surface = Color(0xFF1F1F27),
    surfaceVariant = Color(0xFF1C1C22),
    onSurface = Color(0xFFF5F5F7),
    onSurfaceVariant = Color(0xFFC7C7CC),
    outlineVariant = Color(0x24FFFFFF),
    error = OwoError
)

@Composable
fun OwoTheme(
    useDynamicColor: Boolean = true, // 支持 Material You 动态色彩
    content: @Composable () -> Unit
) {
    val context = LocalContext.current
    val dark = isSystemInDarkTheme()
    val colors = when {
        useDynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            if (dark) dynamicDarkColorScheme(context) else dynamicLightColorScheme(context)
        }
        dark -> OwoDarkColors
        else -> OwoLightColors
    }

    MaterialTheme(
        colorScheme = colors,
        typography = Typography(), // 使用 MaterialTheme.typography；文本单位为 sp，跟随系统 fontScale
        shapes = Shapes(
            small = ShapeDefaults.Small,
            medium = ShapeDefaults.Medium,
            large = ShapeDefaults.Large
        ),
        content = content
    )
}
```

### 鸿蒙 ArkTS / ArkUI

```ts
// EntryAbility.ets
import { UIAbility } from '@kit.AbilityKit';
import { window } from '@kit.ArkUI';

export default class EntryAbility extends UIAbility {
  onWindowStageCreate(windowStage: window.WindowStage): void {
    // WindowStage：用于设置沉浸式、安全区和窗口能力，满足交付文档三端适配要求
    windowStage.getMainWindow((err, win) => {
      if (!err) {
        win.setWindowLayoutFullScreen(true);
        win.setWindowBackgroundColor('#00000000');
      }
    });
    windowStage.loadContent('pages/HomePage');
  }
}

// OwoTokens.ets
export const OwoSize = {
  pagePadding: 24,       // 鸿蒙页面左右边距 24vp
  cardRadius: 16,
  buttonRadius: 14,
  sheetRadius: 28,
  minTouch: 48           // 鸿蒙最小点击区域 48×48vp
}

// 颜色建议在 resources/base/element/color.json 与 dark 资源中维护。
// 使用 Resource：$r('app.color.owo_brand') / $r('app.color.owo_bg')，支持暗黑模式自动切换。
```

## 1. 首页 / 主页面布局（顶部导航、底部导航、列表）

### iOS SwiftUI

```swift
import SwiftUI

struct OwoHomePage: View {
    @Environment(\.colorScheme) private var scheme // ColorScheme：暗黑模式适配
    @Environment(\.dynamicTypeSize) private var dynamicType // DynamicType：动态字体适配
    @State private var selectedTab = "home"
    @State private var selectedCategory = "全部"

    private let categories = ["全部", "大学", "搞笑", "职场", "科幻", "奇幻", "科学科普"]
    private let posts = Array(0..<12)

    var body: some View {
        NavigationStack {
            ZStack {
                OwoColor.appBackground(scheme).ignoresSafeArea()

                ScrollView {
                    LazyVStack(spacing: 16) {
                        categoryTabs
                        LazyVGrid(columns: [GridItem(.flexible()), GridItem(.flexible())], spacing: 12) {
                            ForEach(posts, id: \.self) { index in
                                WhiteBoxCard(index: index)
                            }
                        }
                        .padding(.horizontal, OwoMetric.pagePadding)
                        .padding(.bottom, 88)
                    }
                }
                .safeAreaInset(edge: .top) {
                    topBar
                        .background(.ultraThinMaterial) // iOS 16+ 半透明材质
                }
                .safeAreaInset(edge: .bottom) {
                    bottomTabBar
                        .background(.ultraThinMaterial)
                }
            }
            .navigationBarHidden(true)
        }
    }

    private var topBar: some View {
        VStack(spacing: 12) {
            HStack {
                Text("鸥我")
                    .font(.largeTitle.bold()) // Large Title 34pt，支持 Dynamic Type
                    .foregroundStyle(OwoColor.primaryText(scheme))
                Spacer()
                Button(action: {}) {
                    Image(systemName: "magnifyingglass") // SF Symbols
                        .font(.system(size: 22, weight: .semibold))
                        .frame(width: OwoMetric.minTouch, height: OwoMetric.minTouch)
                }
                .tint(OwoColor.primaryText(scheme))
            }
            .padding(.horizontal, OwoMetric.pagePadding)
        }
        .padding(.top, 4)
        .padding(.bottom, 8)
    }

    private var categoryTabs: some View {
        ScrollView(.horizontal, showsIndicators: false) {
            HStack(spacing: 24) {
                ForEach(categories, id: \.self) { item in
                    Button(item) { selectedCategory = item }
                        .font(.subheadline.weight(selectedCategory == item ? .bold : .regular))
                        .foregroundStyle(selectedCategory == item ? OwoColor.brand : OwoColor.secondaryText(scheme))
                        .frame(minHeight: OwoMetric.minTouch) // 点击热区 ≥44pt
                }
            }
            .padding(.horizontal, OwoMetric.pagePadding)
        }
    }

    private var bottomTabBar: some View {
        HStack {
            tab("home", "house.fill", "首页")
            tab("explore", "sparkles", "探索")
            tab("contacts", "bubble.left.and.bubble.right.fill", "通讯录")
            tab("avatar", "person.crop.circle.badge.sparkles", "分身")
            tab("profile", "person.crop.circle.fill", "我的")
        }
        .frame(height: 49) // iOS Tab Bar 49pt，不含 Home Indicator
        .padding(.horizontal, 8)
    }

    private func tab(_ id: String, _ icon: String, _ title: String) -> some View {
        Button { selectedTab = id } label: {
            VStack(spacing: 3) {
                Image(systemName: icon)
                    .font(.system(size: 25)) // Tab 图标 25×25pt
                Text(title).font(.caption2)
            }
            .frame(maxWidth: .infinity, minHeight: OwoMetric.minTouch)
            .foregroundStyle(selectedTab == id ? OwoColor.brand : OwoColor.secondaryText(scheme))
        }
    }
}

struct WhiteBoxCard: View {
    @Environment(\.colorScheme) private var scheme
    let index: Int

    var body: some View {
        VStack(alignment: .leading, spacing: 10) {
            HStack {
                Circle().fill(OwoColor.brand.opacity(0.16)).frame(width: 28, height: 28)
                Spacer()
                Image(systemName: "heart")
                    .frame(width: 44, height: 44) // 图标视觉小，命中区 44pt
            }
            Text("AI 白盒内容标题 \(index + 1)")
                .font(.headline) // Headline 17pt，Dynamic Type
                .lineLimit(2)
            Text("这里展示内容摘要，最多三行，超出后省略。")
                .font(.subheadline)
                .foregroundStyle(OwoColor.secondaryText(scheme))
                .lineLimit(3)
        }
        .padding(16)
        .background(OwoColor.card(scheme))
        .clipShape(RoundedRectangle(cornerRadius: OwoMetric.cardRadius))
        .shadow(color: .black.opacity(scheme == .dark ? 0.28 : 0.04), radius: 12, x: 0, y: 2)
    }
}
```

### Android Jetpack Compose

```kotlin
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.grid.*
import androidx.compose.foundation.horizontalScroll
import androidx.compose.foundation.rememberScrollState
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.text.style.TextOverflow
import androidx.compose.ui.unit.dp

@Composable
fun OwoHomePage() {
    OwoTheme(useDynamicColor = true) {
        var selectedTab by remember { mutableStateOf("home") }
        var selectedCategory by remember { mutableStateOf("全部") }
        val categories = listOf("全部", "大学", "搞笑", "职场", "科幻", "奇幻", "科学科普")

        Scaffold(
            modifier = Modifier
                .fillMaxSize()
                .windowInsetsPadding(WindowInsets.safeDrawing), // WindowInsets：适配状态栏、挖孔、底部手势区
            topBar = {
                TopAppBar(
                    title = {
                        Text(
                            "鸥我",
                            style = MaterialTheme.typography.headlineMedium, // Material 动态字体，单位 sp
                            fontWeight = FontWeight.Bold
                        )
                    },
                    actions = {
                        IconButton(
                            onClick = {},
                            modifier = Modifier.size(48.dp) // 48×48dp，符合 Material 最小触摸目标
                        ) {
                            Icon(Icons.Default.Search, contentDescription = "搜索")
                        }
                    }
                )
            },
            bottomBar = {
                NavigationBar(
                    tonalElevation = 3.dp,
                    windowInsets = NavigationBarDefaults.windowInsets
                ) {
                    listOf(
                        Triple("home", Icons.Default.Home, "首页"),
                        Triple("explore", Icons.Default.AutoAwesome, "探索"),
                        Triple("contacts", Icons.Default.Chat, "通讯录"),
                        Triple("avatar", Icons.Default.Person, "分身"),
                        Triple("profile", Icons.Default.AccountCircle, "我的")
                    ).forEach { item ->
                        NavigationBarItem(
                            selected = selectedTab == item.first,
                            onClick = { selectedTab = item.first },
                            icon = { Icon(item.second, contentDescription = item.third) }, // 24×24dp Material 图标
                            label = { Text(item.third, style = MaterialTheme.typography.labelSmall) }
                        )
                    }
                }
            }
        ) { padding ->
            Column(Modifier.padding(padding)) {
                Row(
                    Modifier
                        .horizontalScroll(rememberScrollState())
                        .padding(horizontal = 16.dp, vertical = 8.dp), // 页面左右边距 16dp
                    horizontalArrangement = Arrangement.spacedBy(24.dp)
                ) {
                    categories.forEach { item ->
                        TextButton(
                            onClick = { selectedCategory = item },
                            modifier = Modifier.heightIn(min = 48.dp) // Tab 点击区 ≥48dp
                        ) {
                            Text(
                                item,
                                color = if (selectedCategory == item) MaterialTheme.colorScheme.primary else MaterialTheme.colorScheme.onSurfaceVariant,
                                fontWeight = if (selectedCategory == item) FontWeight.Bold else FontWeight.Normal
                            )
                        }
                    }
                }

                LazyVerticalGrid(
                    columns = GridCells.Fixed(2),
                    contentPadding = PaddingValues(start = 16.dp, end = 16.dp, bottom = 16.dp),
                    horizontalArrangement = Arrangement.spacedBy(12.dp),
                    verticalArrangement = Arrangement.spacedBy(12.dp)
                ) {
                    items(12) { index -> WhiteBoxCard(index) }
                }
            }
        }
    }
}

@Composable
fun WhiteBoxCard(index: Int) {
    ElevatedCard(
        shape = MaterialTheme.shapes.large, // 卡片圆角约 12-16dp
        elevation = CardDefaults.elevatedCardElevation(defaultElevation = 1.dp),
        colors = CardDefaults.elevatedCardColors(containerColor = MaterialTheme.colorScheme.surface)
    ) {
        Column(Modifier.padding(16.dp), verticalArrangement = Arrangement.spacedBy(10.dp)) {
            Row(Modifier.fillMaxWidth(), horizontalArrangement = Arrangement.SpaceBetween) {
                Surface(color = MaterialTheme.colorScheme.primary.copy(alpha = 0.16f), shape = MaterialTheme.shapes.large) {
                    Spacer(Modifier.size(28.dp))
                }
                IconButton(onClick = {}, modifier = Modifier.size(48.dp)) {
                    Icon(Icons.Default.FavoriteBorder, contentDescription = "点赞")
                }
            }
            Text(
                "AI 白盒内容标题 ${index + 1}",
                style = MaterialTheme.typography.titleMedium,
                fontWeight = FontWeight.SemiBold,
                maxLines = 2,
                overflow = TextOverflow.Ellipsis
            )
            Text(
                "这里展示内容摘要，最多三行，超出后省略。",
                style = MaterialTheme.typography.bodyMedium,
                color = MaterialTheme.colorScheme.onSurfaceVariant,
                maxLines = 3,
                overflow = TextOverflow.Ellipsis
            )
        }
    }
}
```

### 鸿蒙 ArkTS

```ts
import { window } from '@kit.ArkUI';

@Entry
@Component
struct OwoHomePage {
  @State selectedTab: string = 'home'
  @State selectedCategory: string = '全部'
  private categories: string[] = ['全部', '大学', '搞笑', '职场', '科幻', '奇幻', '科学科普']

  build() {
    Column() {
      this.TopBar()
      this.CategoryTabs()

      Grid() {
        ForEach([0, 1, 2, 3, 4, 5, 6, 7], (index: number) => {
          GridItem() {
            WhiteBoxCard({ index: index })
          }
        })
      }
      .columnsTemplate('1fr 1fr')
      .columnsGap(12)
      .rowsGap(12)
      .padding({ left: 24, right: 24, bottom: 88 }) // 鸿蒙页面边距 24vp
      .layoutWeight(1)

      this.BottomTabs()
    }
    .width('100%')
    .height('100%')
    .backgroundColor($r('app.color.owo_bg')) // Resource：支持浅色/深色资源自动切换
    .fontFamily('HarmonyOS Sans') // HarmonyOS Sans 字体
    .expandSafeArea([SafeAreaType.SYSTEM], [SafeAreaEdge.TOP, SafeAreaEdge.BOTTOM]) // SafeArea：避让状态栏和底部手势区
  }

  @Builder TopBar() {
    Row() {
      Text('鸥我')
        .fontSize(32) // 页面标题 32fp，跟随系统字体缩放
        .fontWeight(FontWeight.Bold)
        .fontColor($r('app.color.owo_text_primary'))
      Blank()
      Button({ type: ButtonType.Circle }) {
        SymbolGlyph($r('sys.symbol.magnifyingglass'))
          .fontSize(24)
      }
      .width(48).height(48) // 48×48vp，符合鸿蒙最小点击区域
      .backgroundColor(Color.Transparent)
    }
    .height(56)
    .padding({ left: 24, right: 24 })
  }

  @Builder CategoryTabs() {
    Scroll() {
      Row({ space: 24 }) {
        ForEach(this.categories, (item: string) => {
          Text(item)
            .fontSize(14)
            .fontWeight(this.selectedCategory === item ? FontWeight.Bold : FontWeight.Regular)
            .fontColor(this.selectedCategory === item ? $r('app.color.owo_brand') : $r('app.color.owo_text_secondary'))
            .height(48) // Tab 点击区 48vp
            .onClick(() => this.selectedCategory = item)
        })
      }.padding({ left: 24, right: 24 })
    }
    .scrollable(ScrollDirection.Horizontal)
    .height(48)
  }

  @Builder BottomTabs() {
    Row() {
      this.TabItem('home', '首页', $r('sys.symbol.house_fill'))
      this.TabItem('explore', '探索', $r('sys.symbol.sparkles'))
      this.TabItem('contacts', '通讯录', $r('sys.symbol.bubble_left_and_bubble_right_fill'))
      this.TabItem('avatar', '分身', $r('sys.symbol.person_crop_circle'))
      this.TabItem('profile', '我的', $r('sys.symbol.person_crop_circle_fill'))
    }
    .height(56)
    .padding({ left: 6, right: 6 })
    .backgroundColor($r('app.color.owo_card'))
  }

  @Builder TabItem(id: string, title: string, icon: Resource) {
    Column({ space: 3 }) {
      SymbolGlyph(icon).fontSize(24)
      Text(title).fontSize(11)
    }
    .fontColor(this.selectedTab === id ? $r('app.color.owo_brand') : $r('app.color.owo_text_tertiary'))
    .layoutWeight(1)
    .height(48)
    .justifyContent(FlexAlign.Center)
    .onClick(() => this.selectedTab = id)
  }
}

@Component
struct WhiteBoxCard {
  @Prop index: number
  build() {
    Column({ space: 10 }) {
      Row() {
        Circle().width(28).height(28).fill($r('app.color.owo_brand_soft'))
        Blank()
        Button({ type: ButtonType.Circle }) {
          SymbolGlyph($r('sys.symbol.heart')).fontSize(22)
        }
        .width(48).height(48)
        .backgroundColor(Color.Transparent)
      }.width('100%')

      Text(`AI 白盒内容标题 ${this.index + 1}`)
        .fontSize(16)
        .fontWeight(FontWeight.Medium)
        .fontColor($r('app.color.owo_text_primary'))
        .maxLines(2)
        .textOverflow({ overflow: TextOverflow.Ellipsis })

      Text('这里展示内容摘要，最多三行，超出后省略。')
        .fontSize(14)
        .fontColor($r('app.color.owo_text_secondary'))
        .maxLines(3)
        .textOverflow({ overflow: TextOverflow.Ellipsis })
    }
    .padding(16)
    .borderRadius(16)
    .backgroundColor($r('app.color.owo_card'))
    .shadow({ radius: 12, offsetY: 2, color: '#0A000000' })
  }
}
```

## 2. 列表页（列表项、下拉刷新、上拉加载、空态、错误态）

### iOS SwiftUI

```swift
import SwiftUI

enum ListState { case loading, empty, error, content }

struct OwoListPage: View {
    @Environment(\.colorScheme) private var scheme
    @Environment(\.dynamicTypeSize) private var dynamicType
    @State private var state: ListState = .content
    @State private var items = Array(0..<20)

    var body: some View {
        NavigationStack {
            Group {
                switch state {
                case .loading:
                    ProgressView("加载中")
                        .font(.body) // Dynamic Type
                case .empty:
                    OwoEmptyState(title: "暂无内容", message: "有新内容后会出现在这里", buttonTitle: "去探索")
                case .error:
                    OwoErrorState(onRetry: { state = .content })
                case .content:
                    List {
                        ForEach(items, id: \.self) { item in
                            OwoListRow(index: item)
                                .listRowInsets(EdgeInsets(top: 8, leading: 16, bottom: 8, trailing: 16))
                                .listRowSeparatorTint(Color(.separator))
                        }
                        if !items.isEmpty {
                            ProgressView()
                                .frame(maxWidth: .infinity)
                                .onAppear { loadMore() } // 上拉加载：滚动到底部触发
                        }
                    }
                    .listStyle(.plain)
                    .refreshable { await refresh() } // iOS 原生下拉刷新
                }
            }
            .background(OwoColor.appBackground(scheme))
            .navigationTitle("列表")
            .safeAreaInset(edge: .bottom) {
                Color.clear.frame(height: 1) // SafeAreaInset：避免列表底部贴 Home Indicator
            }
        }
    }

    private func refresh() async { items = Array(0..<20) }
    private func loadMore() { items.append(contentsOf: items.count..<(items.count + 10)) }
}

struct OwoListRow: View {
    @Environment(\.colorScheme) private var scheme
    let index: Int

    var body: some View {
        HStack(spacing: 12) {
            Circle()
                .fill(OwoColor.brand.opacity(0.18))
                .frame(width: 48, height: 48)
            VStack(alignment: .leading, spacing: 4) {
                Text("列表项标题 \(index)")
                    .font(.headline)
                    .lineLimit(1)
                Text("副标题最多两行，超出省略，跟随系统字体缩放。")
                    .font(.subheadline)
                    .foregroundStyle(OwoColor.secondaryText(scheme))
                    .lineLimit(2)
            }
            Spacer()
            Image(systemName: "chevron.right") // SF Symbols
                .foregroundStyle(OwoColor.secondaryText(scheme))
        }
        .frame(minHeight: 56) // 列表行高 ≥44pt
        .contentShape(Rectangle())
    }
}

struct OwoErrorState: View {
    let onRetry: () -> Void
    var body: some View {
        VStack(spacing: 12) {
            Image(systemName: "wifi.exclamationmark")
                .font(.system(size: 48))
                .foregroundStyle(OwoColor.error)
            Text("网络开小差了")
                .font(.headline)
            Text("请检查网络后重试")
                .font(.subheadline)
                .foregroundStyle(.secondary)
            Button("重试", action: onRetry)
                .font(.body.weight(.semibold))
                .frame(minWidth: 120, minHeight: OwoMetric.minTouch)
                .background(OwoColor.brand)
                .foregroundStyle(.white)
                .clipShape(RoundedRectangle(cornerRadius: OwoMetric.buttonRadius))
        }
        .padding(32)
    }
}
```

### Android Jetpack Compose

```kotlin
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.*
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.ChevronRight
import androidx.compose.material.icons.filled.WifiOff
import androidx.compose.material3.*
import androidx.compose.material3.pulltorefresh.PullToRefreshBox
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.text.style.TextOverflow
import androidx.compose.ui.unit.dp

sealed interface OwoListState {
    data object Loading : OwoListState
    data object Empty : OwoListState
    data object Error : OwoListState
    data object Content : OwoListState
}

@Composable
fun OwoListPage() {
    OwoTheme {
        var state by remember { mutableStateOf<OwoListState>(OwoListState.Content) }
        var refreshing by remember { mutableStateOf(false) }
        val items = remember { mutableStateListOf<Int>().apply { addAll(0 until 20) } }

        Scaffold(
            modifier = Modifier
                .fillMaxSize()
                .windowInsetsPadding(WindowInsets.safeDrawing), // WindowInsets 安全区
            topBar = { TopAppBar(title = { Text("列表", style = MaterialTheme.typography.titleLarge) }) }
        ) { padding ->
            Box(Modifier.padding(padding).fillMaxSize()) {
                when (state) {
                    OwoListState.Loading -> CircularProgressIndicator(Modifier.align(androidx.compose.ui.Alignment.Center))
                    OwoListState.Empty -> OwoEmptyState(title = "暂无内容", message = "有新内容后会出现在这里")
                    OwoListState.Error -> OwoErrorState(onRetry = { state = OwoListState.Content })
                    OwoListState.Content -> PullToRefreshBox(
                        isRefreshing = refreshing,
                        onRefresh = { refreshing = true; refreshing = false } // 下拉刷新
                    ) {
                        LazyColumn(
                            contentPadding = PaddingValues(16.dp),
                            verticalArrangement = Arrangement.spacedBy(8.dp)
                        ) {
                            items(items) { item ->
                                OwoListRow(item)
                            }
                            item {
                                LaunchedEffect(Unit) {
                                    // 上拉加载：列表底部 item 出现时请求下一页
                                    items.addAll(items.size until items.size + 10)
                                }
                                CircularProgressIndicator(
                                    modifier = Modifier.fillMaxWidth().padding(16.dp).wrapContentWidth(),
                                    strokeWidth = 3.dp
                                )
                            }
                        }
                    }
                }
            }
        }
    }
}

@Composable
fun OwoListRow(index: Int) {
    ListItem(
        modifier = Modifier
            .fillMaxWidth()
            .heightIn(min = 56.dp), // 列表行高 ≥48dp，满足 Material 点击目标
        leadingContent = {
            Surface(
                shape = MaterialTheme.shapes.large,
                color = MaterialTheme.colorScheme.primary.copy(alpha = 0.18f),
                modifier = Modifier.size(48.dp)
            ) {}
        },
        headlineContent = {
            Text(
                "列表项标题 $index",
                style = MaterialTheme.typography.titleMedium,
                fontWeight = FontWeight.SemiBold,
                maxLines = 1,
                overflow = TextOverflow.Ellipsis
            )
        },
        supportingContent = {
            Text(
                "副标题最多两行，超出省略，跟随系统 fontScale。",
                style = MaterialTheme.typography.bodyMedium,
                color = MaterialTheme.colorScheme.onSurfaceVariant,
                maxLines = 2,
                overflow = TextOverflow.Ellipsis
            )
        },
        trailingContent = { Icon(Icons.Default.ChevronRight, contentDescription = null) }
    )
}

@Composable
fun OwoErrorState(onRetry: () -> Unit) {
    Column(
        modifier = Modifier.fillMaxSize().padding(32.dp),
        verticalArrangement = Arrangement.Center,
        horizontalAlignment = androidx.compose.ui.Alignment.CenterHorizontally
    ) {
        Icon(Icons.Default.WifiOff, contentDescription = null, tint = MaterialTheme.colorScheme.error, modifier = Modifier.size(48.dp))
        Spacer(Modifier.height(12.dp))
        Text("网络开小差了", style = MaterialTheme.typography.titleMedium)
        Text("请检查网络后重试", style = MaterialTheme.typography.bodyMedium, color = MaterialTheme.colorScheme.onSurfaceVariant)
        Spacer(Modifier.height(16.dp))
        Button(
            onClick = onRetry,
            modifier = Modifier.heightIn(min = 48.dp) // 按钮点击区域 ≥48dp
        ) {
            Text("重试")
        }
    }
}
```

### 鸿蒙 ArkTS

```ts
@Component
struct OwoListPage {
  @State listState: string = 'content'
  @State refreshing: boolean = false
  @State items: number[] = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

  build() {
    Column() {
      Row() {
        Text('列表')
          .fontSize(20)
          .fontWeight(FontWeight.Bold)
          .fontColor($r('app.color.owo_text_primary'))
      }
      .height(56)
      .padding({ left: 24, right: 24 })

      if (this.listState === 'loading') {
        LoadingProgress().width(40).height(40).layoutWeight(1)
      } else if (this.listState === 'empty') {
        OwoEmptyState({ title: '暂无内容', message: '有新内容后会出现在这里' })
      } else if (this.listState === 'error') {
        OwoErrorState({ onRetry: () => this.listState = 'content' })
      } else {
        Refresh({ refreshing: $$this.refreshing }) {
          List({ space: 8 }) {
            ForEach(this.items, (item: number) => {
              ListItem() {
                OwoListRow({ index: item })
              }
            })
            ListItem() {
              LoadingProgress().width(32).height(32).margin(16)
            }
          }
          .padding({ left: 24, right: 24, bottom: 24 })
          .onReachEnd(() => {
            // 上拉加载：列表触底后追加数据
            this.items = this.items.concat([this.items.length, this.items.length + 1, this.items.length + 2])
          })
        }
        .onRefreshing(() => {
          // 下拉刷新：鸿蒙 Refresh 组件
          this.refreshing = false
        })
      }
    }
    .width('100%')
    .height('100%')
    .backgroundColor($r('app.color.owo_bg'))
    .fontFamily('HarmonyOS Sans')
    .expandSafeArea([SafeAreaType.SYSTEM], [SafeAreaEdge.TOP, SafeAreaEdge.BOTTOM])
  }
}

@Component
struct OwoListRow {
  @Prop index: number

  build() {
    Row({ space: 12 }) {
      Circle()
        .width(48)
        .height(48)
        .fill($r('app.color.owo_brand_soft'))
      Column({ space: 4 }) {
        Text(`列表项标题 ${this.index}`)
          .fontSize(16)
          .fontWeight(FontWeight.Medium)
          .maxLines(1)
          .textOverflow({ overflow: TextOverflow.Ellipsis })
        Text('副标题最多两行，超出省略，跟随系统字体缩放。')
          .fontSize(14)
          .fontColor($r('app.color.owo_text_secondary'))
          .maxLines(2)
          .textOverflow({ overflow: TextOverflow.Ellipsis })
      }.layoutWeight(1)
      SymbolGlyph($r('sys.symbol.chevron_right')).fontSize(18)
    }
    .height(64) // 行高 64vp，满足最小 48vp 点击目标
    .padding({ left: 16, right: 16 })
    .backgroundColor($r('app.color.owo_card'))
    .borderRadius(16)
  }
}
```

## 3. 详情页（图片、文字、按钮、分享）

### iOS SwiftUI

```swift
import SwiftUI

struct OwoDetailPage: View {
    @Environment(\.colorScheme) private var scheme
    @Environment(\.dynamicTypeSize) private var dynamicType
    @Environment(\.dismiss) private var dismiss
    @State private var followed = false
    @State private var showShare = false

    var body: some View {
        ScrollView {
            VStack(alignment: .leading, spacing: 16) {
                ZStack(alignment: .bottomLeading) {
                    RoundedRectangle(cornerRadius: 0)
                        .fill(LinearGradient(colors: [OwoColor.brand.opacity(0.38), OwoColor.brandStrong.opacity(0.24)], startPoint: .topLeading, endPoint: .bottomTrailing))
                        .frame(height: 220)
                    Circle()
                        .fill(OwoColor.brand.opacity(0.18))
                        .frame(width: 88, height: 88)
                        .overlay(Image(systemName: "person.fill").font(.system(size: 40)))
                        .padding(16)
                }

                VStack(alignment: .leading, spacing: 12) {
                    Text("Alex Chen")
                        .font(.title.bold()) // Title 28pt，Dynamic Type
                    Text("CEO / AI 面试官 / 创业者")
                        .font(.subheadline)
                        .foregroundStyle(OwoColor.secondaryText(scheme))
                    Text("这里展示人物简介、咨询服务、正在寻人信息和 AI 分身能力说明。正文支持动态字体，长文本自动换行。")
                        .font(.body)
                        .lineSpacing(4)

                    HStack(spacing: 12) {
                        Button(followed ? "已关注" : "关注") { followed.toggle() }
                            .frame(maxWidth: .infinity, minHeight: OwoMetric.minTouch)
                            .background(followed ? OwoColor.brand.opacity(0.12) : OwoColor.brand)
                            .foregroundStyle(followed ? OwoColor.brand : .white)
                            .clipShape(RoundedRectangle(cornerRadius: 999))

                        Button("立即开聊") {}
                            .frame(maxWidth: .infinity, minHeight: OwoMetric.minTouch)
                            .background(OwoColor.brand)
                            .foregroundStyle(.white)
                            .clipShape(RoundedRectangle(cornerRadius: 999))
                    }
                }
                .padding(.horizontal, OwoMetric.pagePadding)
            }
        }
        .background(OwoColor.appBackground(scheme))
        .safeAreaInset(edge: .top) {
            HStack {
                Button { dismiss() } label: {
                    Image(systemName: "chevron.left")
                        .frame(width: OwoMetric.minTouch, height: OwoMetric.minTouch)
                }
                Spacer()
                Button { showShare = true } label: {
                    Image(systemName: "square.and.arrow.up") // SF Symbols 分享
                        .frame(width: OwoMetric.minTouch, height: OwoMetric.minTouch)
                }
            }
            .padding(.horizontal, 8)
            .background(.ultraThinMaterial)
        }
        .sheet(isPresented: $showShare) {
            Text("分享面板")
                .font(.headline)
                .presentationDetents([.medium])
                .presentationDragIndicator(.visible)
        }
    }
}
```

### Android Jetpack Compose

```kotlin
import androidx.compose.foundation.background
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.rememberScrollState
import androidx.compose.foundation.shape.CircleShape
import androidx.compose.foundation.verticalScroll
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.ArrowBack
import androidx.compose.material.icons.filled.Person
import androidx.compose.material.icons.filled.Share
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.dp

@Composable
fun OwoDetailPage(onBack: () -> Unit) {
    OwoTheme {
        var followed by remember { mutableStateOf(false) }
        var showShare by remember { mutableStateOf(false) }

        Scaffold(
            modifier = Modifier.windowInsetsPadding(WindowInsets.safeDrawing),
            topBar = {
                TopAppBar(
                    title = {},
                    navigationIcon = {
                        IconButton(onClick = onBack, modifier = Modifier.size(48.dp)) {
                            Icon(Icons.Default.ArrowBack, contentDescription = "返回")
                        }
                    },
                    actions = {
                        IconButton(onClick = { showShare = true }, modifier = Modifier.size(48.dp)) {
                            Icon(Icons.Default.Share, contentDescription = "分享")
                        }
                    }
                )
            }
        ) { padding ->
            Column(
                Modifier
                    .padding(padding)
                    .verticalScroll(rememberScrollState())
            ) {
                Box(
                    Modifier
                        .fillMaxWidth()
                        .height(220.dp)
                        .background(MaterialTheme.colorScheme.primary.copy(alpha = 0.22f))
                ) {
                    Surface(
                        modifier = Modifier.padding(16.dp).size(88.dp).align(androidx.compose.ui.Alignment.BottomStart),
                        shape = CircleShape,
                        color = MaterialTheme.colorScheme.primary.copy(alpha = 0.18f)
                    ) {
                        Icon(Icons.Default.Person, contentDescription = null, modifier = Modifier.padding(22.dp))
                    }
                }

                Column(Modifier.padding(16.dp), verticalArrangement = Arrangement.spacedBy(12.dp)) {
                    Text("Alex Chen", style = MaterialTheme.typography.headlineMedium, fontWeight = FontWeight.Bold)
                    Text("CEO / AI 面试官 / 创业者", style = MaterialTheme.typography.bodyMedium, color = MaterialTheme.colorScheme.onSurfaceVariant)
                    Text(
                        "这里展示人物简介、咨询服务、正在寻人信息和 AI 分身能力说明。正文支持 fontScale，长文本自动换行。",
                        style = MaterialTheme.typography.bodyLarge
                    )
                    Row(horizontalArrangement = Arrangement.spacedBy(12.dp)) {
                        OutlinedButton(
                            onClick = { followed = !followed },
                            modifier = Modifier.weight(1f).heightIn(min = 48.dp) // 按钮触摸目标 ≥48dp
                        ) { Text(if (followed) "已关注" else "关注") }

                        Button(
                            onClick = {},
                            modifier = Modifier.weight(1f).heightIn(min = 48.dp)
                        ) { Text("立即开聊") }
                    }
                }
            }
        }

        if (showShare) {
            ModalBottomSheet(onDismissRequest = { showShare = false }) {
                Text("分享面板", style = MaterialTheme.typography.titleLarge, modifier = Modifier.padding(24.dp))
                Spacer(Modifier.height(24.dp + WindowInsets.navigationBars.asPaddingValues().calculateBottomPadding()))
            }
        }
    }
}
```

### 鸿蒙 ArkTS

```ts
@Component
struct OwoDetailPage {
  @State followed: boolean = false
  @State showShare: boolean = false

  build() {
    Stack() {
      Scroll() {
        Column({ space: 16 }) {
          Stack({ alignContent: Alignment.BottomStart }) {
            Rect()
              .width('100%')
              .height(220)
              .fill($r('app.color.owo_brand_soft'))
            Circle()
              .width(88)
              .height(88)
              .fill($r('app.color.owo_brand_soft'))
              .overlay(SymbolGlyph($r('sys.symbol.person_fill')).fontSize(40))
              .margin(16)
          }

          Column({ space: 12 }) {
            Text('Alex Chen')
              .fontSize(28)
              .fontWeight(FontWeight.Bold)
              .fontColor($r('app.color.owo_text_primary'))
            Text('CEO / AI 面试官 / 创业者')
              .fontSize(14)
              .fontColor($r('app.color.owo_text_secondary'))
            Text('这里展示人物简介、咨询服务、正在寻人信息和 AI 分身能力说明。正文支持系统字体缩放，长文本自动换行。')
              .fontSize(16)
              .lineHeight(24)

            Row({ space: 12 }) {
              Button(this.followed ? '已关注' : '关注')
                .height(48) // 按钮高度 48vp，符合鸿蒙最小触摸目标
                .layoutWeight(1)
                .borderRadius(999)
                .backgroundColor(this.followed ? $r('app.color.owo_brand_soft') : $r('app.color.owo_brand'))
                .fontColor(this.followed ? $r('app.color.owo_brand') : Color.White)
                .onClick(() => this.followed = !this.followed)

              Button('立即开聊')
                .height(48)
                .layoutWeight(1)
                .borderRadius(999)
                .backgroundColor($r('app.color.owo_brand'))
                .fontColor(Color.White)
            }
          }.padding({ left: 24, right: 24 })
        }
      }

      Row() {
        Button({ type: ButtonType.Circle }) {
          SymbolGlyph($r('sys.symbol.chevron_left')).fontSize(22)
        }.width(48).height(48).backgroundColor(Color.Transparent)
        Blank()
        Button({ type: ButtonType.Circle }) {
          SymbolGlyph($r('sys.symbol.square_and_arrow_up')).fontSize(22)
        }
        .width(48).height(48)
        .backgroundColor(Color.Transparent)
        .onClick(() => this.showShare = true)
      }
      .height(56)
      .padding({ left: 8, right: 8 })
      .align(Alignment.Top)
      .backgroundColor($r('app.color.owo_card'))

      if (this.showShare) {
        OwoShareSheet({ onClose: () => this.showShare = false })
      }
    }
    .backgroundColor($r('app.color.owo_bg'))
    .fontFamily('HarmonyOS Sans')
    .expandSafeArea([SafeAreaType.SYSTEM], [SafeAreaEdge.TOP, SafeAreaEdge.BOTTOM])
  }
}
```

## 4. 弹窗 / 对话框（确认、输入、底部弹窗）

### iOS SwiftUI

```swift
import SwiftUI

struct OwoDialogDemo: View {
    @Environment(\.colorScheme) private var scheme
    @State private var showConfirm = false
    @State private var showInput = false
    @State private var showSheet = false
    @State private var text = ""

    var body: some View {
        VStack(spacing: 16) {
            Button("确认弹窗") { showConfirm = true }
            Button("输入弹窗") { showInput = true }
            Button("底部弹窗") { showSheet = true }
        }
        .font(.body) // Dynamic Type
        .frame(maxWidth: .infinity, maxHeight: .infinity)
        .background(OwoColor.appBackground(scheme))
        .safeAreaInset(edge: .bottom) { Color.clear.frame(height: 1) }
        .alert("确认删除？", isPresented: $showConfirm) {
            Button("取消", role: .cancel) {}
            Button("删除", role: .destructive) {}
        } message: {
            Text("删除后不可恢复。") // iOS Alert：危险操作红色按钮
        }
        .alert("输入名称", isPresented: $showInput) {
            TextField("请输入", text: $text)
            Button("取消", role: .cancel) {}
            Button("确定") {}
        }
        .sheet(isPresented: $showSheet) {
            VStack(spacing: 16) {
                Capsule().fill(Color.secondary.opacity(0.3)).frame(width: 36, height: 5)
                Text("底部弹窗")
                    .font(.headline)
                Button("完成") { showSheet = false }
                    .frame(maxWidth: .infinity, minHeight: OwoMetric.minTouch)
                    .background(OwoColor.brand)
                    .foregroundStyle(.white)
                    .clipShape(RoundedRectangle(cornerRadius: OwoMetric.buttonRadius))
            }
            .padding(24)
            .presentationDetents([.medium, .large])
            .presentationDragIndicator(.visible)
        }
    }
}
```

### Android Jetpack Compose

```kotlin
import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp

@Composable
fun OwoDialogDemo() {
    OwoTheme {
        var showConfirm by remember { mutableStateOf(false) }
        var showInput by remember { mutableStateOf(false) }
        var showSheet by remember { mutableStateOf(false) }
        var text by remember { mutableStateOf("") }

        Column(
            Modifier
                .fillMaxSize()
                .windowInsetsPadding(WindowInsets.safeDrawing)
                .padding(16.dp),
            verticalArrangement = Arrangement.Center
        ) {
            Button(onClick = { showConfirm = true }, modifier = Modifier.fillMaxWidth().heightIn(min = 48.dp)) { Text("确认弹窗") }
            Spacer(Modifier.height(12.dp))
            Button(onClick = { showInput = true }, modifier = Modifier.fillMaxWidth().heightIn(min = 48.dp)) { Text("输入弹窗") }
            Spacer(Modifier.height(12.dp))
            Button(onClick = { showSheet = true }, modifier = Modifier.fillMaxWidth().heightIn(min = 48.dp)) { Text("底部弹窗") }
        }

        if (showConfirm) {
            AlertDialog(
                onDismissRequest = { showConfirm = false },
                shape = MaterialTheme.shapes.extraLarge, // Dialog 圆角 28dp
                title = { Text("确认删除？", style = MaterialTheme.typography.titleLarge) },
                text = { Text("删除后不可恢复。", style = MaterialTheme.typography.bodyMedium) },
                confirmButton = { TextButton(onClick = { showConfirm = false }) { Text("删除", color = MaterialTheme.colorScheme.error) } },
                dismissButton = { TextButton(onClick = { showConfirm = false }) { Text("取消") } }
            )
        }

        if (showInput) {
            AlertDialog(
                onDismissRequest = { showInput = false },
                title = { Text("输入名称") },
                text = {
                    OutlinedTextField(
                        value = text,
                        onValueChange = { text = it },
                        label = { Text("请输入") },
                        singleLine = true
                    )
                },
                confirmButton = { TextButton(onClick = { showInput = false }) { Text("确定") } },
                dismissButton = { TextButton(onClick = { showInput = false }) { Text("取消") } }
            )
        }

        if (showSheet) {
            ModalBottomSheet(
                onDismissRequest = { showSheet = false },
                shape = MaterialTheme.shapes.extraLarge // Bottom Sheet 顶部圆角 28dp
            ) {
                Text("底部弹窗", style = MaterialTheme.typography.titleLarge, modifier = Modifier.padding(24.dp))
                Button(
                    onClick = { showSheet = false },
                    modifier = Modifier.fillMaxWidth().padding(horizontal = 24.dp).heightIn(min = 48.dp)
                ) { Text("完成") }
                Spacer(Modifier.height(24.dp + WindowInsets.navigationBars.asPaddingValues().calculateBottomPadding()))
            }
        }
    }
}
```

### 鸿蒙 ArkTS

```ts
@Component
struct OwoDialogDemo {
  @State showConfirm: boolean = false
  @State showInput: boolean = false
  @State showSheet: boolean = false
  @State inputText: string = ''

  build() {
    Column({ space: 12 }) {
      Button('确认弹窗').height(48).width('100%').onClick(() => this.showConfirm = true)
      Button('输入弹窗').height(48).width('100%').onClick(() => this.showInput = true)
      Button('底部弹窗').height(48).width('100%').onClick(() => this.showSheet = true)
    }
    .padding(24)
    .justifyContent(FlexAlign.Center)
    .width('100%')
    .height('100%')
    .backgroundColor($r('app.color.owo_bg'))
    .fontFamily('HarmonyOS Sans')
    .expandSafeArea([SafeAreaType.SYSTEM], [SafeAreaEdge.TOP, SafeAreaEdge.BOTTOM])
    .bindContentCover($$this.showSheet, this.BottomSheetBuilder(), {
      modalTransition: ModalTransition.DEFAULT
    })
    .bindMenu(this.showConfirm, {
      builder: this.ConfirmDialogBuilder()
    })
  }

  @Builder ConfirmDialogBuilder() {
    Column({ space: 12 }) {
      Text('确认删除？').fontSize(20).fontWeight(FontWeight.Bold)
      Text('删除后不可恢复。').fontSize(14).fontColor($r('app.color.owo_text_secondary'))
      Row({ space: 12 }) {
        Button('取消').height(48).layoutWeight(1).onClick(() => this.showConfirm = false)
        Button('删除')
          .height(48)
          .layoutWeight(1)
          .backgroundColor($r('app.color.owo_error'))
          .fontColor(Color.White)
          .onClick(() => this.showConfirm = false)
      }
    }
    .padding(24)
    .borderRadius(24) // 鸿蒙 Dialog 圆角 24-28vp
    .backgroundColor($r('app.color.owo_card'))
  }

  @Builder InputDialogBuilder() {
    Column({ space: 12 }) {
      Text('输入名称').fontSize(20).fontWeight(FontWeight.Bold)
      TextInput({ placeholder: '请输入', text: this.inputText })
        .height(48)
        .borderRadius(12)
        .onChange((value: string) => this.inputText = value)
      Button('确定').height(48).width('100%').onClick(() => this.showInput = false)
    }
    .padding(24)
    .backgroundColor($r('app.color.owo_card'))
    .borderRadius(24)
  }

  @Builder BottomSheetBuilder() {
    Column({ space: 16 }) {
      Rect().width(36).height(5).borderRadius(999).fill('#33000000')
      Text('底部弹窗').fontSize(20).fontWeight(FontWeight.Bold)
      Button('完成')
        .height(48) // 底部弹窗按钮点击区 48vp
        .width('100%')
        .borderRadius(14)
        .onClick(() => this.showSheet = false)
    }
    .padding({ left: 24, right: 24, top: 12, bottom: 24 })
    .backgroundColor($r('app.color.owo_card'))
    .borderRadius({ topLeft: 28, topRight: 28 })
  }
}
```

## 5. 空状态页面（占位图、提示文字、操作按钮）

### iOS SwiftUI

```swift
import SwiftUI

struct OwoEmptyState: View {
    @Environment(\.colorScheme) private var scheme
    let title: String
    let message: String
    var buttonTitle: String = "去看看"

    var body: some View {
        VStack(spacing: 12) {
            Image(systemName: "tray")
                .font(.system(size: 56, weight: .regular)) // SF Symbols 空态图标
                .foregroundStyle(OwoColor.brand.opacity(0.75))
                .frame(width: 96, height: 96) // 空态占位 96×96
            Text(title)
                .font(.headline)
                .foregroundStyle(OwoColor.primaryText(scheme))
            Text(message)
                .font(.subheadline)
                .foregroundStyle(OwoColor.secondaryText(scheme))
                .multilineTextAlignment(.center)
                .lineLimit(2)
            Button(buttonTitle) {}
                .font(.body.weight(.semibold))
                .frame(minWidth: 128, minHeight: OwoMetric.minTouch)
                .background(OwoColor.brand)
                .foregroundStyle(.white)
                .clipShape(RoundedRectangle(cornerRadius: OwoMetric.buttonRadius))
                .padding(.top, 4)
        }
        .padding(32)
        .frame(maxWidth: .infinity, maxHeight: .infinity)
        .background(OwoColor.appBackground(scheme))
        .safeAreaInset(edge: .bottom) { Color.clear.frame(height: 1) }
    }
}
```

### Android Jetpack Compose

```kotlin
import androidx.compose.foundation.layout.*
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Inbox
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.style.TextAlign
import androidx.compose.ui.unit.dp

@Composable
fun OwoEmptyState(
    title: String = "暂无内容",
    message: String = "有新内容后会出现在这里",
    actionText: String = "去看看",
    onAction: () -> Unit = {}
) {
    OwoTheme {
        Column(
            modifier = Modifier
                .fillMaxSize()
                .windowInsetsPadding(WindowInsets.safeDrawing)
                .padding(32.dp),
            verticalArrangement = Arrangement.Center,
            horizontalAlignment = androidx.compose.ui.Alignment.CenterHorizontally
        ) {
            Icon(
                Icons.Default.Inbox,
                contentDescription = null,
                tint = MaterialTheme.colorScheme.primary.copy(alpha = 0.75f),
                modifier = Modifier.size(96.dp) // 空态占位 96×96dp
            )
            Spacer(Modifier.height(12.dp))
            Text(title, style = MaterialTheme.typography.titleMedium)
            Text(
                message,
                style = MaterialTheme.typography.bodyMedium,
                color = MaterialTheme.colorScheme.onSurfaceVariant,
                textAlign = TextAlign.Center
            )
            Spacer(Modifier.height(16.dp))
            Button(
                onClick = onAction,
                modifier = Modifier.heightIn(min = 48.dp) // 按钮高度 48dp，符合 Material 最小触摸目标
            ) {
                Text(actionText)
            }
        }
    }
}
```

### 鸿蒙 ArkTS

```ts
@Component
struct OwoEmptyState {
  @Prop title: string = '暂无内容'
  @Prop message: string = '有新内容后会出现在这里'

  build() {
    Column({ space: 12 }) {
      SymbolGlyph($r('sys.symbol.tray'))
        .fontSize(56)
        .fontColor($r('app.color.owo_brand'))
        .width(96)
        .height(96) // 空态占位 96×96vp
      Text(this.title)
        .fontSize(17)
        .fontWeight(FontWeight.Medium)
        .fontColor($r('app.color.owo_text_primary'))
      Text(this.message)
        .fontSize(14)
        .fontColor($r('app.color.owo_text_secondary'))
        .textAlign(TextAlign.Center)
        .maxLines(2)
      Button('去看看')
        .height(48) // 按钮高度 48vp
        .padding({ left: 20, right: 20 })
        .borderRadius(14)
        .backgroundColor($r('app.color.owo_brand'))
        .fontColor(Color.White)
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
    .padding(32)
    .backgroundColor($r('app.color.owo_bg'))
    .fontFamily('HarmonyOS Sans')
    .expandSafeArea([SafeAreaType.SYSTEM], [SafeAreaEdge.TOP, SafeAreaEdge.BOTTOM])
  }
}
```

## 6. 设置页（列表、开关、分组）

### iOS SwiftUI

```swift
import SwiftUI

struct OwoSettingsPage: View {
    @Environment(\.colorScheme) private var scheme
    @Environment(\.dynamicTypeSize) private var dynamicType
    @State private var notifications = true
    @State private var selectedAppearance = "跟随系统"

    var body: some View {
        NavigationStack {
            List {
                Section("外观") {
                    Picker("外观设置", selection: $selectedAppearance) {
                        Text("跟随系统").tag("跟随系统")
                        Text("浅色").tag("浅色")
                        Text("深色").tag("深色")
                    }
                    .pickerStyle(.segmented) // 只保留系统/浅色/深色，不考虑暖色/冷色
                    .padding(.vertical, 8)
                }

                Section("通知与隐私") {
                    Toggle(isOn: $notifications) {
                        Label("消息通知", systemImage: "bell") // SF Symbols
                    }
                    .frame(minHeight: OwoMetric.minTouch) // 设置行命中区 ≥44pt

                    NavigationLink {
                        Text("隐私设置").font(.body)
                    } label: {
                        Label("隐私设置", systemImage: "lock")
                            .font(.body)
                    }
                    .frame(minHeight: OwoMetric.minTouch)
                }

                Section("账号安全") {
                    NavigationLink {
                        Text("修改密码").font(.body)
                    } label: {
                        Label("修改密码", systemImage: "eye")
                    }
                    .frame(minHeight: OwoMetric.minTouch)
                }
            }
            .scrollContentBackground(.hidden)
            .background(OwoColor.appBackground(scheme))
            .navigationTitle("设置")
            .safeAreaInset(edge: .bottom) { Color.clear.frame(height: 1) } // 安全区适配
        }
    }
}
```

### Android Jetpack Compose

```kotlin
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.dp

@Composable
fun OwoSettingsPage() {
    OwoTheme(useDynamicColor = true) {
        var notifications by remember { mutableStateOf(true) }
        var appearance by remember { mutableStateOf("system") }

        Scaffold(
            modifier = Modifier.windowInsetsPadding(WindowInsets.safeDrawing),
            topBar = { TopAppBar(title = { Text("设置", style = MaterialTheme.typography.titleLarge) }) }
        ) { padding ->
            LazyColumn(
                modifier = Modifier.padding(padding).fillMaxSize(),
                contentPadding = PaddingValues(16.dp),
                verticalArrangement = Arrangement.spacedBy(16.dp)
            ) {
                item {
                    SettingsGroup(title = "外观") {
                        SingleChoiceSegmentedButtonRow(Modifier.fillMaxWidth()) {
                            listOf("system" to "跟随系统", "light" to "浅色", "dark" to "深色").forEachIndexed { index, pair ->
                                SegmentedButton(
                                    selected = appearance == pair.first,
                                    onClick = { appearance = pair.first },
                                    shape = SegmentedButtonDefaults.itemShape(index = index, count = 3)
                                ) {
                                    Text(pair.second, style = MaterialTheme.typography.labelLarge)
                                }
                            }
                        }
                        // 外观设置只包含系统/浅色/深色；暗黑模式由 MaterialTheme.colorScheme 承载
                    }
                }

                item {
                    SettingsGroup(title = "通知与隐私") {
                        SettingsSwitchRow(
                            icon = Icons.Default.Notifications,
                            title = "消息通知",
                            checked = notifications,
                            onCheckedChange = { notifications = it }
                        )
                        SettingsNavRow(icon = Icons.Default.Lock, title = "隐私设置")
                    }
                }

                item {
                    SettingsGroup(title = "账号安全") {
                        SettingsNavRow(icon = Icons.Default.Visibility, title = "修改密码")
                    }
                }
            }
        }
    }
}

@Composable
fun SettingsGroup(title: String, content: @Composable ColumnScope.() -> Unit) {
    Column(verticalArrangement = Arrangement.spacedBy(8.dp)) {
        Text(title, style = MaterialTheme.typography.labelLarge, color = MaterialTheme.colorScheme.onSurfaceVariant)
        ElevatedCard(
            shape = MaterialTheme.shapes.large,
            colors = CardDefaults.elevatedCardColors(containerColor = MaterialTheme.colorScheme.surface),
            elevation = CardDefaults.elevatedCardElevation(defaultElevation = 1.dp)
        ) {
            Column(Modifier.padding(8.dp), content = content)
        }
    }
}

@Composable
fun SettingsSwitchRow(
    icon: androidx.compose.ui.graphics.vector.ImageVector,
    title: String,
    checked: Boolean,
    onCheckedChange: (Boolean) -> Unit
) {
    ListItem(
        modifier = Modifier.heightIn(min = 56.dp), // 设置行高 ≥48dp
        leadingContent = { Icon(icon, contentDescription = null) },
        headlineContent = { Text(title, style = MaterialTheme.typography.bodyLarge) },
        trailingContent = { Switch(checked = checked, onCheckedChange = onCheckedChange) }
    )
}

@Composable
fun SettingsNavRow(icon: androidx.compose.ui.graphics.vector.ImageVector, title: String) {
    ListItem(
        modifier = Modifier.heightIn(min = 56.dp),
        leadingContent = { Icon(icon, contentDescription = null) },
        headlineContent = { Text(title, style = MaterialTheme.typography.bodyLarge) },
        trailingContent = { Icon(Icons.Default.ChevronRight, contentDescription = null) }
    )
}
```

### 鸿蒙 ArkTS

```ts
@Component
struct OwoSettingsPage {
  @State notifications: boolean = true
  @State appearance: string = 'system'

  build() {
    Column() {
      Row() {
        Text('设置')
          .fontSize(20)
          .fontWeight(FontWeight.Bold)
          .fontColor($r('app.color.owo_text_primary'))
      }
      .height(56)
      .padding({ left: 24, right: 24 })

      Scroll() {
        Column({ space: 16 }) {
          this.Group('外观', () => {
            Row({ space: 8 }) {
              this.AppearanceButton('system', '跟随系统')
              this.AppearanceButton('light', '浅色')
              this.AppearanceButton('dark', '深色')
            }
            .padding(8)
            // 外观设置仅系统/浅色/深色，不考虑暖色/冷色；颜色通过 Resource dark 资源切换
          })

          this.Group('通知与隐私', () => {
            this.SwitchRow($r('sys.symbol.bell'), '消息通知')
            this.NavRow($r('sys.symbol.lock'), '隐私设置')
          })

          this.Group('账号安全', () => {
            this.NavRow($r('sys.symbol.eye'), '修改密码')
          })
        }
        .padding({ left: 24, right: 24, bottom: 24 })
      }
      .layoutWeight(1)
    }
    .width('100%')
    .height('100%')
    .backgroundColor($r('app.color.owo_bg'))
    .fontFamily('HarmonyOS Sans')
    .expandSafeArea([SafeAreaType.SYSTEM], [SafeAreaEdge.TOP, SafeAreaEdge.BOTTOM])
  }

  @Builder Group(title: string, content: () => void) {
    Column({ space: 8 }) {
      Text(title)
        .fontSize(13)
        .fontColor($r('app.color.owo_text_secondary'))
      Column() {
        content()
      }
      .padding(8)
      .borderRadius(16)
      .backgroundColor($r('app.color.owo_card'))
      .shadow({ radius: 12, offsetY: 2, color: '#0A000000' })
    }
  }

  @Builder AppearanceButton(value: string, label: string) {
    Button(label)
      .height(48) // 分段按钮点击区 48vp
      .layoutWeight(1)
      .borderRadius(999)
      .backgroundColor(this.appearance === value ? $r('app.color.owo_brand') : $r('app.color.owo_brand_soft'))
      .fontColor(this.appearance === value ? Color.White : $r('app.color.owo_brand'))
      .onClick(() => this.appearance = value)
  }

  @Builder SwitchRow(icon: Resource, title: string) {
    Row({ space: 12 }) {
      SymbolGlyph(icon).fontSize(24)
      Text(title).fontSize(16).layoutWeight(1)
      Toggle({ type: ToggleType.Switch, isOn: this.notifications })
        .width(51)
        .height(31) // Switch 视觉 51×31，外层行高保证 48vp 点击区
        .onChange((value: boolean) => this.notifications = value)
    }
    .height(56)
    .padding({ left: 8, right: 8 })
  }

  @Builder NavRow(icon: Resource, title: string) {
    Row({ space: 12 }) {
      SymbolGlyph(icon).fontSize(24)
      Text(title).fontSize(16).layoutWeight(1)
      SymbolGlyph($r('sys.symbol.chevron_right')).fontSize(18)
    }
    .height(56) // 设置行高 ≥48vp
    .padding({ left: 8, right: 8 })
  }
}
```
