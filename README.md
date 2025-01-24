# Modüler MVVM Geliştirme Dokümanı
**Versiyon:** 2.0

## İçindekiler

17. [Memory Management](#17-memory-management)
    - Memory Leak Önleme
    - Image Loading ve Caching

18. [Performance Optimization](#18-performance-optimization)
    - ANR Önleme
    - Startup Optimization

19. [Security](#19-security)
    - Encryption
    - Proguard Configuration

20. [Testing](#20-testing)
    - Integration Testing
    - End-to-End Test
    - Test Doubles
    - Screenshot Testing
    - Performance Testing

21. [CI/CD Entegrasyonu](#21-cicd-entegrasyonu)
    - GitHub Actions Workflow
    - Fastlane Entegrasyonu

22. [Module Bazlı Performance Monitoring](#22-module-bazlı-performance-monitoring)
    - Firebase Performance Monitoring
    - Custom Performance Metrics

23. [Security Best Practices](#23-security-best-practices)
    - Sensitive Data Handling
    - Network Security Configuration
    - Code Obfuscation

24. [Module'ler Arası Navigation Contract'lar](#24-moduleler-arası-navigation-contractlar)
    - Feature Module Navigation
    - Deep Link Handling

25. [Safe Args Kullanımı Detayları](#25-safe-args-kullanımı-detayları)
    - Navigation ile Safe Args
    - Modüller Arası Safe Args

26. [Memory Leak Önleme Stratejileri](#26-memory-leak-önleme-stratejileri)
    - View Binding Memory Leak
    - ViewModel Memory Leak
    - Coroutine Memory Leak

27. [Large Object Handling](#27-large-object-handling)
    - Bitmap ve Büyük Resimler
    - Büyük Liste Verileri

28. [Garbage Collection Optimizasyonları](#28-garbage-collection-optimizasyonları)
    - Object Pool Pattern
    - Weak Reference Kullanımı

29. [Önemli Kaynaklar ve Referanslar](#29-önemli-kaynaklar-ve-referanslar)
    - Resmi Kaynaklar
    - En İyi Pratikler
    - Performans Optimizasyonu
    - Test Stratejileri
    - Güvenlik

### Ekler
- Kod Örnekleri
- Şemalar ve Diyagramlar
- Best Practices Kontrol Listesi
- Troubleshooting Rehberi


## 17. Memory Management

### A. Memory Leak Önleme

```kotlin
class BaseActivity : AppCompatActivity() {
    private var _binding: ViewBinding? = null
    private val binding get() = _binding!!

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        _binding = // binding initialization
    }

    override fun onDestroy() {
        super.onDestroy()
        _binding = null // Prevent memory leak
    }
}
```

### B. Image Loading ve Caching

```kotlin
// Coil kullanım örneği
class ImageLoaderManager @Inject constructor(
    private val context: Context
) {
    private val imageLoader = ImageLoader.Builder(context)
        .memoryCache {
            MemoryCache.Builder(context)
                .maxSizePercent(0.25) // Use 25% of app memory
                .build()
        }
        .diskCache {
            DiskCache.Builder()
                .directory(context.cacheDir.resolve("image_cache"))
                .maxSizePercent(0.02) // Use 2% of disk space
                .build()
        }
        .build()

    fun loadImage(
        imageUrl: String,
        imageView: ImageView,
        placeholder: Drawable? = null
    ) {
        imageView.load(imageUrl, imageLoader) {
            crossfade(true)
            placeholder(placeholder)
            error(R.drawable.ic_error)
        }
    }
}
```

## 18. Performance Optimization

### A. ANR Önleme

```kotlin
class UserRepository @Inject constructor(
    private val api: UserApi,
    private val db: UserDatabase,
    private val dispatcher: CoroutineDispatcher = Dispatchers.IO
) {
    suspend fun getUser(id: String): User = withContext(dispatcher) {
        db.userDao().getUser(id) ?: api.getUser(id).also { user ->
            db.userDao().insertUser(user)
        }
    }
}
```

### B. Startup Optimization

```kotlin
// Lazy initialization örneği
class MyApplication : Application() {
    private val userManager by lazy {
        UserManager(this)
    }

    override fun onCreate() {
        super.onCreate()
        
        // Startup işlemlerini ContentProvider ile yönetme
        AppInitializer.getInstance(this)
            .initializeComponent(WorkManagerInitializer::class.java)
    }
}
```

## 19. Security

### A. Encryption

```kotlin
class EncryptionManager @Inject constructor(context: Context) {
    private val masterKey = MasterKey.Builder(context)
        .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
        .build()

    private val encryptedSharedPreferences = EncryptedSharedPreferences.create(
        context,
        "secret_shared_prefs",
        masterKey,
        EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
        EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
    )

    fun saveSecureData(key: String, value: String) {
        encryptedSharedPreferences.edit()
            .putString(key, value)
            .apply()
    }

    fun getSecureData(key: String): String? =
        encryptedSharedPreferences.getString(key, null)
}
```

### B. Proguard Configuration

```proguard
# proguard-rules.pro
-keepclassmembers class * extends androidx.lifecycle.ViewModel {
    <init>(...);
}

-keepclassmembers class * extends com.example.app.data.model.* {
    <init>(...);
}

# Retrofit rules
-keepattributes Signature
-keepattributes *Annotation*
-keep class retrofit2.** { *; }
```

## 20. Testing

### A. Integration Testing

```kotlin
@HiltAndroi

### A. İsimlendirme
- [ ] Paket isimleri küçük harf ve noktalı gösterimde
- [ ] Sınıflar PascalCase ile isimlendirildi
- [ ] Değişken ve fonksiyonlar camelCase ile isimlendirildi
- [ ] Layout dosyaları activity_, fragment_, item_ ile başlıyor

### B. Kotlin Standartları
- [ ] Nullable değişkenler ? ile işaretlendi
- [ ] Data class'lar immutable (val) property'ler kullanıyor
- [ ] Sealed class'lar UI state yönetimi için kullanıldı
- [ ] Coroutine ve Flow doğru şekilde entegre edildi

## 20. Testing (Devam)

### A. Integration Testing

```kotlin
@HiltAndroidTest
class UserFlowTest {
    @get:Rule
    val hiltRule = HiltAndroidRule(this)

    @Inject
    lateinit var userRepository: UserRepository

    @Test
    fun testUserProfileUpdate() = runBlocking {
        // Given
        val user = User("1", "test@example.com")
        userRepository.saveUser(user)

        // When
        val updatedName = "Updated Name"
        userRepository.updateUserName("1", updatedName)

        // Then
        val result = userRepository.getUser("1")
        assertEquals(updatedName, result.name)
    }
}
```

### B. End-to-End Test Stratejisi

```kotlin
@LargeTest
@RunWith(AndroidJUnit4::class)
class UserProfileEndToEndTest {
    @get:Rule
    val activityRule = ActivityScenarioRule(MainActivity::class.java)

    @Test
    fun testProfileUpdateFlow() {
        // Login
        onView(withId(R.id.emailInput))
            .perform(typeText("test@example.com"))
        onView(withId(R.id.passwordInput))
            .perform(typeText("password123"))
        onView(withId(R.id.loginButton))
            .perform(click())

        // Navigate to Profile
        onView(withId(R.id.profileMenuItem))
            .perform(click())

        // Update Profile
        onView(withId(R.id.nameInput))
            .perform(typeText("New Name"))
        onView(withId(R.id.saveButton))
            .perform(click())

        // Verify Update
        onView(withId(R.id.profileName))
            .check(matches(withText("New Name")))
    }
}
```

### C. Test Doubles

```kotlin
// Mock
@Test
fun `test user profile update with mock`() = runBlocking {
    // Given
    val mockRepository = mock<UserRepository>()
    whenever(mockRepository.updateUser(any()))
        .thenReturn(Result.success(Unit))

    val useCase = UpdateUserProfileUseCase(mockRepository)

    // When
    val result = useCase.invoke(UpdateUserProfileParams("1", "New Name"))

    // Then
    verify(mockRepository).updateUser(any())
    assert(result.isSuccess)
}

// Fake
class FakeUserRepository : UserRepository {
    private val users = mutableMapOf<String, User>()

    override suspend fun getUser(id: String): User? {
        return users[id]
    }

    override suspend fun saveUser(user: User) {
        users[user.id] = user
    }
}

// Stub
class UserRepositoryStub : UserRepository {
    override suspend fun getUser(id: String): User {
        return User("1", "Test User", "test@example.com")
    }
}
```

### D. Screenshot Testing

```kotlin
class ProfileScreenshotTest {
    @get:Rule
    val composeRule = createComposeRule()

    @Test
    fun captureProfileScreen() {
        composeRule.setContent {
            ProfileScreen(
                state = ProfileScreenState(
                    name = "John Doe",
                    email = "john@example.com",
                    avatarUrl = "https://example.com/avatar.jpg"
                )
            )
        }

        composeRule.onRoot().captureToImage().assertAgainstGolden()
    }
}
```

### E. Performance Testing

```kotlin
@RunWith(AndroidJUnit4::class)
class AppStartupPerfTest {
    @get:Rule
    val benchmarkRule = BenchmarkRule()

    @Test
    fun measureAppStartup() {
        benchmarkRule.measureRepeated {
            pressHome()
            startActivityAndWait()
        }
    }
}

// Custom Performance Test
class DatabasePerformanceTest {
    @Test
    fun measureBulkInsert() = runBlocking {
        val startTime = System.nanoTime()
        
        // Perform bulk insert
        userDao.insertAll(generateLargeUserList())
        
        val endTime = System.nanoTime()
        val duration = (endTime - startTime) / 1_000_000 // Convert to milliseconds
        
        assertTrue("Bulk insert took too long", duration < 1000) // Should complete under 1 second
    }
}
```

## 21. CI/CD Entegrasyonu

### A. GitHub Actions Workflow

```yaml
name: Android CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    
    - name: Set up JDK
      uses: actions/setup-java@v2
      with:
        java-version: '17'
        distribution: 'adopt'
        
    - name: Grant execute permission for gradlew
      run: chmod +x gradlew
      
    - name: Run tests
      run: ./gradlew test
      
    - name: Build Debug APK
      run: ./gradlew assembleDebug
      
    - name: Upload APK
      uses: actions/upload-artifact@v2
      with:
        name: app-debug
        path: app/build/outputs/apk/debug/app-debug.apk
```

### B. Fastlane Entegrasyonu

```ruby
# fastlane/Fastfile
default_platform(:android)

platform :android do
  desc "Run all tests"
  lane :test do
    gradle(task: "test")
  end

  desc "Deploy a new beta build to Firebase App Distribution"
  lane :beta do
    gradle(task: "clean assembleRelease")
    firebase_app_distribution(
        app: "1:123456789:android:abcd1234",
        groups: "testers",
        release_notes: "Bug fixes and improvements"
    )
  end

  desc "Deploy a new version to the Google Play"
  lane :deploy do
    gradle(task: "clean bundleRelease")
    upload_to_play_store(
      track: "production",
      release_status: "completed",
      aab: "../app/build/outputs/bundle/release/app-release.aab"
    )
  end
end
```

## 22. Module Bazlı Performance Monitoring

### A. Firebase Performance Monitoring

```kotlin
class PerformanceMonitoringManager @Inject constructor() {
    private val firebasePerformance = FirebasePerformance.getInstance()

    fun startTrace(traceName: String): Trace {
        return firebasePerformance.newTrace(traceName)
    }

    fun recordNetworkRequest(url: String, responseTime: Long) {
        val metric = firebasePerformance.newHttpMetric(url, FirebasePerformance.HttpMethod.GET)
        metric.setResponsePayloadSize(responseSize)
        metric.markResponseStart()
        Thread.sleep(responseTime) // Simulate network request
        metric.markResponseEnd()
        metric.stop()
    }
}

// Kullanım örneği
class UserRepository @Inject constructor(
    private val performanceMonitor: PerformanceMonitoringManager
) {
    suspend fun getUser(id: String): User {
        val trace = performanceMonitor.startTrace("get_user")
        try {
            trace.putAttribute("user_id", id)
            val result = api.getUser(id)
            trace.putMetric("response_size", result.toString().length.toLong())
            return result
        } finally {
            trace.stop()
        }
    }
}
```

### B. Custom Performance Metrics

```kotlin
class ModulePerformanceTracker {
    private val metrics = mutableMapOf<String, Long>()
    
    fun trackOperation(moduleName: String, operationName: String, block: () -> Unit) {
        val startTime = System.nanoTime()
        block()
        val endTime = System.nanoTime()
        
        val duration = (endTime - startTime) / 1_000_000 // ms cinsinden
        val key = "$moduleName:$operationName"
        metrics[key] = duration
    }
    
    fun getModuleMetrics(moduleName: String): Map<String, Long> {
        return metrics.filterKeys { it.startsWith(moduleName) }
    }
    
    fun resetMetrics() {
        metrics.clear()
    }
}

// Kullanım
class FeatureModule {
    private val performanceTracker = ModulePerformanceTracker()
    
    fun performOperation() {
        performanceTracker.trackOperation("feature_module", "data_load") {
            // Operation logic
        }
    }
}
```

## 23. Security Best Practices

### A. Sensitive Data Handling

```kotlin
class SecureDataManager @Inject constructor(context: Context) {
    private val keyStore = KeyStore.getInstance("AndroidKeyStore").apply {
        load(null)
    }
    
    private val encryptedPreferences = EncryptedSharedPreferences.create(
        context,
        "secure_prefs",
        getMasterKey(context),
        EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
        EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
    )
    
    private fun getMasterKey(context: Context): MasterKey {
        return MasterKey.Builder(context)
            .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
            .setUserAuthenticationRequired(true)
            .setUserAuthenticationValidityDurationSeconds(30)
            .build()
    }
    
    fun saveSecureData(key: String, value: String) {
        encryptedPreferences.edit().putString(key, value).apply()
    }
    
    fun getSecureData(key: String): String? {
        return encryptedPreferences.getString(key, null)
    }
}
```

### B. Network Security Configuration

```xml
<!-- network_security_config.xml -->
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <debug-overrides>
        <trust-anchors>
            <certificates src="system"/>
            <certificates src="user"/>
        </trust-anchors>
    </debug-overrides>
    
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system"/>
        </trust-anchors>
    </base-config>
    
    <domain-config>
        <domain includeSubdomains="true">example.com</domain>
        <pin-set expiration="2024-01-01">
            <pin digest="SHA-256">7HIpactkIAq2Y49orFOOQKurWxmmSFZhBCoQYcRhJ3Y=</pin>
            <!-- Backup pin -->
            <pin digest="SHA-256">fwza0LRMXouZHRC8Ei+4PyuldPDcf3UKgO/04cDM1oE=</pin>
        </pin-set>
    </domain-config>
</network-security-config>
```

### C. Code Obfuscation (ProGuard/R8)

```proguard
# proguard-rules.pro

# Keep model classes
-keep class com.example.app.domain.model.** { *; }

# Keep API interfaces
-keep interface com.example.app.data.api.** { *; }

# Keep custom annotations
-keep @com.example.app.annotation.** class * { *; }

# Remove logging in release
-assumenosideeffects class android.util.Log {
    public static *** d(...);
    public static *** v(...);
    public static *** i(...);
}

# Keep Retrofit service methods
-keepclassmembernames interface * {
    @retrofit2.http.* <methods>;
}

# Keep Kotlin Serialization
-keepattributes *Annotation*, InnerClasses
-dontnote kotlinx.serialization.AnnotationsKt
```

## 24. Module'ler Arası Navigation Contract'lar

### A. Feature Module Navigation

```kotlin
// Core module'de navigation contract
interface ProfileNavigationContract {
    fun navigateToProfile(userId: String)
    fun navigateToSettings()
    fun navigateToEditProfile(userId: String)
}

// Feature module'de implementation
class ProfileNavigationImpl @Inject constructor(
    private val navController: NavController
) : ProfileNavigationContract {
    override fun navigateToProfile(userId: String) {
        navController.navigate(
            ProfileFragmentDirections.actionToProfile(userId)
        )
    }
    
    override fun navigateToSettings() {
        navController.navigate(R.id.settingsFragment)
    }
    
    override fun navigateToEditProfile(userId: String) {
        navController.navigate(
            ProfileFragmentDirections.actionToEditProfile(userId)
        )
    }
}

// Dependency injection
@Module
@InstallIn(SingletonComponent::class)
object NavigationModule {
    @Provides
    fun provideProfileNavigation(
        navController: NavController
    ): ProfileNavigationContract = ProfileNavigationImpl(navController)
}
```

### B. Deep Link Handling

```kotlin
// DeepLinkHandler.kt
class DeepLinkHandler @Inject constructor(
    private val navController: NavController
) {
    fun handleDeepLink(deepLinkUri: Uri) {
        when {
            deepLinkUri.isProfileDeepLink() -> handleProfileDeepLink(deepLinkUri)
            deepLinkUri.isProductDeepLink() -> handleProductDeepLink(deepLinkUri)
            else -> handleUnknownDeepLink(deepLinkUri)
        }
    }

    private fun Uri.isProfileDeepLink(): Boolean =
        host == "example.com" && path?.startsWith("/profile") == true

    private fun handleProfileDeepLink(uri: Uri) {
        val userId = uri.getQueryParameter("id") ?: return
        navController.navigate(
            ProfileFragmentDirections.actionToProfile(userId)
        )
    }
}

// Activity'de kullanım
class MainActivity : AppCompatActivity() {
    @Inject
    lateinit var deepLinkHandler: DeepLinkHandler

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        handleIntent(intent)
    }

    override fun onNewIntent(intent: Intent?) {
        super.onNewIntent(intent)
        handleIntent(intent)
    }

    private fun handleIntent(intent: Intent?) {
        intent?.data?.let { uri ->
            deepLinkHandler.handleDeepLink(uri)
        }
    }
}
```

Bu doküman modüler MVVM mimarisinin detaylı bir şekilde uygulanması için gerekli tüm bileşenleri ve en iyi pratikleri içermektedir. Her bir bölüm kendi içinde bağımsız olarak çalışabilir ve projenin ihtiyaçlarına göre uyarlanabilir durumdadır.

Önemli Kaynaklar ve Referanslar:
- [Android Developer Guides](https://developer.android.com/guide

## 25. Safe Args Kullanımı Detayları

### A. Navigation ile Safe Args

```kotlin
// navigation/nav_graph.xml
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    
    <fragment
        android:id="@+id/productListFragment"
        android:name=".presentation.ProductListFragment">
        
        <action
            android:id="@+id/action_to_productDetail"
            app:destination="@id/productDetailFragment">
            <argument
                android:name="productId"
                app:argType="string" />
            <argument
                android:name="productName"
                app:argType="string"
                app:nullable="true" />
            <argument
                android:name="isFromSearch"
                app:argType="boolean"
                android:defaultValue="false" />
        </action>
    </fragment>
</navigation>

// ProductListFragment.kt
class ProductListFragment : Fragment() {
    private fun navigateToDetail(product: Product) {
        val directions = ProductListFragmentDirections
            .actionToProductDetail(
                productId = product.id,
                productName = product.name,
                isFromSearch = viewModel.isSearchActive
            )
        findNavController().navigate(directions)
    }
}

// ProductDetailFragment.kt
@AndroidEntryPoint
class ProductDetailFragment : Fragment() {
    private val args: ProductDetailFragmentArgs by navArgs()
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        val productId = args.productId
        val productName = args.productName
        val isFromSearch = args.isFromSearch
        
        viewModel.loadProductDetails(productId)
    }
}
```

### B. Modüller Arası Safe Args

```kotlin
// feature-product/src/main/res/navigation/product_graph.xml
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/product_graph"
    app:startDestination="@id/productListFragment">
    
    <include app:graph="@navigation/cart_graph" />
    
    <fragment
        android:id="@+id/productListFragment"
        android:name=".ProductListFragment">
        
        <deepLink
            android:id="@+id/deepLink"
            app:uri="example://products/{productId}" />
            
        <argument
            android:name="productId"
            app:argType="string" />
    </fragment>
</navigation>

// build.gradle.kts (feature-product)
plugins {
    id("androidx.navigation.safeargs.kotlin")
}
```

## 26. Memory Leak Önleme Stratejileri

### A. View Binding Memory Leak Önleme

```kotlin
abstract class BaseFragment<VB : ViewBinding> : Fragment() {
    private var _binding: VB? = null
    protected val binding get() = _binding!!
    
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        _binding = inflateBinding(inflater, container)
        return binding.root
    }
    
    abstract fun inflateBinding(inflater: LayoutInflater, container: ViewGroup?): VB
    
    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }
}

// Kullanım örneği
class UserProfileFragment : BaseFragment<FragmentUserProfileBinding>() {
    override fun inflateBinding(
        inflater: LayoutInflater,
        container: ViewGroup?
    ) = FragmentUserProfileBinding.inflate(inflater, container, false)
}
```

### B. ViewModel Memory Leak Önleme

```kotlin
class UserViewModel @Inject constructor(
    private val getUserUseCase: GetUserUseCase
) : ViewModel() {
    private var job: Job? = null
    
    fun loadUser(userId: String) {
        // Önceki job'ı iptal et
        job?.cancel()
        
        job = viewModelScope.launch {
            try {
                val result = getUserUseCase(userId)
                // Handle result
            } finally {
                job = null
            }
        }
    }
    
    override fun onCleared() {
        super.onCleared()
        job?.cancel()
    }
}
```

### C. Coroutine Memory Leak Önleme

```kotlin
abstract class CoroutineScopeFragment : Fragment() {
    private var _coroutineScope: CoroutineScope? = null
    protected val coroutineScope get() = _coroutineScope!!
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        _coroutineScope = CoroutineScope(
            SupervisorJob() + Dispatchers.Main + CoroutineExceptionHandler { _, throwable ->
                handleError(throwable)
            }
        )
    }
    
    override fun onDestroy() {
        super.onDestroy()
        _coroutineScope?.cancel()
        _coroutineScope = null
    }
    
    protected open fun handleError(throwable: Throwable) {
        // Error handling logic
    }
}
```

## 27. Large Object Handling

### A. Bitmap ve Büyük Resimlerin Yönetimi

```kotlin
class ImageManager @Inject constructor(
    private val context: Context
) {
    fun decodeSampledBitmap(
        uri: Uri,
        reqWidth: Int,
        reqHeight: Int
    ): Bitmap? {
        return context.contentResolver.openInputStream(uri)?.use { input ->
            // İlk olarak boyutları oku
            val options = BitmapFactory.Options().apply {
                inJustDecodeBounds = true
            }
            BitmapFactory.decodeStream(input, null, options)
            
            // Örnek boyut hesapla
            options.apply {
                inSampleSize = calculateInSampleSize(this, reqWidth, reqHeight)
                inJustDecodeBounds = false
            }
            
            // Bitmap'i yeniden oku ve döndür
            context.contentResolver.openInputStream(uri)?.use { input2 ->
                BitmapFactory.decodeStream(input2, null, options)
            }
        }
    }
    
    private fun calculateInSampleSize(
        options: BitmapFactory.Options,
        reqWidth: Int,
        reqHeight: Int
    ): Int {
        val (height: Int, width: Int) = options.run { outHeight to outWidth }
        var inSampleSize = 1
        
        if (height > reqHeight || width > reqWidth) {
            val halfHeight: Int = height / 2
            val halfWidth: Int = width / 2
            
            while (halfHeight / inSampleSize >= reqHeight && 
                   halfWidth / inSampleSize >= reqWidth) {
                inSampleSize *= 2
            }
        }
        
        return inSampleSize
    }
}
```

### B. Büyük Liste Verileri Yönetimi

```kotlin
class PaginatedListManager<T> @Inject constructor(
    private val coroutineScope: CoroutineScope
) {
    private val _items = MutableStateFlow<List<T>>(emptyList())
    val items: StateFlow<List<T>> = _items.asStateFlow()
    
    private var isLoading = false
    private var currentPage = 0
    private var hasMoreItems = true
    
    fun loadNextPage(
        pageSize: Int,
        loadPage: suspend (page: Int, size: Int) -> List<T>
    ) {
        if (isLoading || !hasMoreItems) return
        
        isLoading = true
        coroutineScope.launch {
            try {
                val newItems = loadPage(currentPage, pageSize)
                if (newItems.isEmpty()) {
                    hasMoreItems = false
                } else {
                    _items.update { currentItems ->
                        currentItems + newItems
                    }
                    currentPage++
                }
            } finally {
                isLoading = false
            }
        }
    }
    
    fun reset() {
        _items.value = emptyList()
        currentPage = 0
        hasMoreItems = true
        isLoading = false
    }
}
```

## 28. Garbage Collection Optimizasyonları

### A. Object Pool Pattern

```kotlin
class ObjectPool<T>(
    private val maxSize: Int,
    private val factory: () -> T,
    private val reset: (T) -> Unit
) {
    private val pool = ArrayDeque<T>(maxSize)
    private val inUse = mutableSetOf<T>()
    
    @Synchronized
    fun acquire(): T {
        return if (pool.isEmpty()) {
            factory().also { inUse.add(it) }
        } else {
            pool.removeFirst().also { inUse.add(it) }
        }
    }
    
    @Synchronized
    fun release(obj: T) {
        if (inUse.remove(obj)) {
            reset(obj)
            if (pool.size < maxSize) {
                pool.addLast(obj)
            }
        }
    }
}

// Kullanım örneği
class BitmapPool @Inject constructor() {
    private val pool = ObjectPool(
        maxSize = 10,
        factory = {
            Bitmap.createBitmap(100, 100, Bitmap.Config.ARGB_8888)
        },
        reset = { bitmap ->
            bitmap.eraseColor(Color.TRANSPARENT)
        }
    )
    
    fun getBitmap(): Bitmap = pool.acquire()
    
    fun returnBitmap(bitmap: Bitmap) {
        pool.release(bitmap)
    }
}
```

### B. Weak Reference Kullanımı

```kotlin
class CacheManager<K, V> {
    private val cache = mutableMapOf<K, WeakReference<V>>()
    
    fun put(key: K, value: V) {
        cache[key] = WeakReference(value)
    }
    
    fun get(key: K): V? {
        return cache[key]?.get()
    }
    
    fun clear() {
        cache.clear()
    }
    
    fun cleanUp() {
        cache.entries.removeAll { entry ->
            entry.value.get() == null
        }
    }
}

// Kullanım örneği
class ImageCache @Inject constructor() {
    private val cache = CacheManager<String, Bitmap>()
    
    fun cacheImage(url: String, bitmap: Bitmap) {
        cache.put(url, bitmap)
    }
    
    fun getImage(url: String): Bitmap? {
        return cache.get(url)
    }
}
```

## 29. Önemli Kaynaklar ve Referanslar

1. Resmi Kaynaklar
- [Android Developers](https://developer.android.com)
- [Kotlin Documentation](https://kotlinlang.org/docs/home.html)
- [Android Architecture Components](https://developer.android.com/topic/libraries/architecture)

2. En İyi Pratikler
- [Android Architecture Blueprints](https://github.com/android/architecture-samples)
- [Google Android Architecture Guides](https://developer.android.com/topic/architecture)
- [Kotlin Coding Conventions](https://kotlinlang.org/docs/coding-conventions.html)

3. Performans Optimizasyonu
- [Android Performance Patterns](https://www.youtube.com/playlist?list=PLWz5rJ2EKKc9CBxr3BVjPTPoDPLdPIFCE)
- [Android Vitals](https://developer.android.com/topic/performance/vitals)

4. Test Stratejileri
- [Android Testing Guide](https://developer.android.com/training/testing)
- [Testing on Android Blog Posts](https://medium.com/androiddevelopers/tagged/testing)

5. Güvenlik
- [Android Security Best Practices](https://developer.android.com/topic/security/best-practices)
- [OWASP Mobile Security Testing Guide](https://owasp.org/www-project-mobile-security-testing-guide/)

Bu doküman sürekli güncellenmekte ve geliştirilmektedir. En güncel Android geliştirme pratikleri ve teknolojileri için yukarıdaki kaynakları düzenli olarak takip etmenizi öneririz.
