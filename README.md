# Sunflower_Hilt(DI)
## [Hilt](https://developer.android.com/training/dependency-injection/hilt-android?hl=ko)

## Hilt Dependency
```Kotlin
//프로젝트 builde.gradle
buildscript {
    ...
    dependencies {
        ...
        classpath 'com.google.dagger:hilt-android-gradle-plugin:2.28-alpha'
    }
}

//앱 builde.gradle
...
apply plugin: 'kotlin-kapt'
apply plugin: 'dagger.hilt.android.plugin'

android {
    ...
}

dependencies {
    implementation "com.google.dagger:hilt-android:2.28-alpha"
    kapt "com.google.dagger:hilt-android-compiler:2.28-alpha"
    
    //test
    kaptAndroidTest "com.google.dagger:hilt-android-compiler:$rootProject.hiltVersion"
    androidTestImplementation "com.google.dagger:hilt-android-testing:$rootProject.hiltVersion"
}
```
## Hilt + Application + @HiltAndroidApp
Hilt를 사용하기 위해서는 Application 클래스를 반드시 @HiltAndroidApp 과 함께 만들어주고 manifest android:name 에 세팅을 해주어야 합니다.

@HiltAndroidApp는 Application 객체의 수명 주기에 연결된 앱의 최상단 부모 컴포넌트이므로 이와 관련한 수명주기와 ApplicationContext 등 같은 종속 항목들을 하위(서브) 컴포넌트들에게 제공할 수 있습니다.

또한 컴포넌트들은 계층으로 이루어져 있으며 하위(서브) 컴포넌트는 상위 컴포넌트의 의존성에 접근할 수 있습니다.

![image](https://user-images.githubusercontent.com/48902047/149494669-6785f185-ce68-475f-8ab0-ecf43c459da4.png)

Sunflower에서 Application 클래스를 살펴보면 다음과 같이 구현되어 있습니다.
```Kotlin
import android.app.Application
import dagger.hilt.android.HiltAndroidApp

@HiltAndroidApp
class MainApplication : Application()
```
## Hilt + View + @AndroidEntryPoint + Component 계층 및 Scope

![image](https://user-images.githubusercontent.com/48902047/149495116-fbf2ed2f-d122-44f7-a205-9c6f5582f25f.png)

SunFlower 는 Jetpack Navigation과 함께 하나의 액티비티로 구현되어 싱글액티비티 디자인(또는 SPA) 지향하고 있습니다.

따라서 GardenActivity 라는 하나의 액티비티만 존재합니다.

@AndroidEntryPoint는 @HiltAndroidApp 설정 후 사용 가능하며 @AndroidEntryPoint 어노테이션이 추가된 안드로이드 클래스에 DI 컨테이너를 추가 해줍니다.

안드로이드 클래스 중 @AndroidEntryPoint는
- Activity
- Fragment
- View
- Service
- BroadcastReceiver 
를 지원합니다.

앞서 Application 클래스에 @HiltAndroidApp으로 애플리케이션 수준인 최상단 컴포넌트를 사용할 수 있게 되었으니 그 하위(서브) Android 클래스들에 @AndroidEntryPoint 를 설정함으로써 Application(상위) -> 안드로이드 클래스(하위, 밑 예시참고) 종속 항목(dependecies)를 제공할 수 있게 해줍니다. 추가로 프래그먼트에 @AndroidEntryPoint를 설정하려면 상위 개념에 해당하는 액티비티에도 @AndroidEntryPoint가 설정 되어 있어야합니다.
```Kotlin
@AndroidEntryPoint
class GalleryFragment : Fragment() {
```
```Kotlin
@AndroidEntryPoint
class GardenFragment : Fragment() {
```
```Kotlin
@AndroidEntryPoint
class HomeViewPagerFragment : Fragment() {
```
```Kotlin
@AndroidEntryPoint
class PlantDetailFragment : Fragment() {
```
```Kotlin
@AndroidEntryPoint
class PlantListFragment : Fragment() {
```
 Hilt 에서는 밑 사진과 같이 안드로이드 클래스를 위한 표준화된 컴포넌트 세트와 스코프를 제공합니다.
![image](https://user-images.githubusercontent.com/48902047/149495831-fa6a8383-f315-42ac-a62a-981b099f9621.png)
![image](https://user-images.githubusercontent.com/48902047/149496272-da9926a1-721c-4da4-b108-67a3f81b4258.png)
![image](https://user-images.githubusercontent.com/48902047/149496331-01193c2b-0ecd-4d32-bc5a-a113f5eee553.png)
![image](https://user-images.githubusercontent.com/48902047/149496349-be2e27f7-c720-40f8-a1fe-3fe1e93e46fa.png)

## Hilt + ViewModel + @HiltViewModel + @Inject constructor()
![image](https://user-images.githubusercontent.com/48902047/149496464-62f6e41f-13e0-4f64-b2d3-212df1d624a6.png)

Hilt가 적용된 ViewModel 입니다.
```Kotlin
@HiltViewModel
class GalleryViewModel @Inject constructor(
    private val repository: UnsplashRepository
) : ViewModel() {
```
```Kotlin
@HiltViewModel
class GardenPlantingListViewModel @Inject internal constructor(
    gardenPlantingRepository: GardenPlantingRepository
) : ViewModel() {
```
```Kotlin
/**
 * The ViewModel for [PlantListFragment].
 */
@HiltViewModel
class PlantListViewModel @Inject internal constructor(
    plantRepository: PlantRepository,
    private val savedStateHandle: SavedStateHandle
) : ViewModel() {
```
```Kotlin
/**
 * The ViewModel used in [PlantDetailFragment].
 */
class PlantDetailViewModel @AssistedInject constructor(
    plantRepository: PlantRepository,
    private val gardenPlantingRepository: GardenPlantingRepository,
    @Assisted private val plantId: String
) : ViewModel() {
```
- @Inject constructor
  - 생성자 삽입 방법으로 클래스의 인스턴스를 제공하는 방법을 Hilt에 알려주게 됩니다.따라서 inject 뒤의 인스턴트를 제공하는 방법도 알아야 합니다.
- @HiltViewModel
  - @HiltViewModel 어노테이션이 붙은 ViewModel은 HiltViewModelFactory에 의해 생성되고 @AndroidEntryPoint 어노테이션이 붙은 액티비티와 프래그먼트에서 기본 디폴트로 회수해오는게 가능해지게 합니다.
  - @HilteViewModel에서 @Inject 어노테이션이 붙은 생성자는 생성자 파라미터가 Hilt에 의해 주입받을 거라는 거라고 정의내리는 종속성을 갖게 해줍니다.

ViewModel은 @Inject와 @HiltViewModel 만 사용하면 됩니다.

@AndroidEntryPoint 어노테이션가 있는 액티비티나 프래그먼트에서 이 Hilt가 적용된 ViewModel 인스턴스를 얻으려면 ViewModelProvider 나 kt-extensions 인 by viewmodels() 을 사용하면 됩니다.
![image](https://user-images.githubusercontent.com/48902047/149498522-22550205-b132-4409-a998-9419487a8475.png)

ViewModel 관련 Hilt에서 위 사항 말고도 다음과 같이 @AssistedInject , @Assisted, @AssistedFactory 라는 어노테이션도 있습니다.

@AssitedInject는 Dagger의 컴파일타임 안정성과 의존성 주입 이후 원하는 의존성을 얻을 수 있도록 도와줍니다. 쉽게 말해서 동적인 파라미터들과 함께 의존성 주입을 할 수 있습니다. (예를들어 SavedState 정보나 고유ID값 등, WorkManager에서도 @Assisted 을 사용합니다. WorkerParameters와 context 의 동적인 전달을 받기 위해서입니다. )

그리고 밑 코드를 보면 지금까지의 ViewModel 처럼 by viewModels()로 생성하는거와 다르게 @AssistedFactory를 사용한다는 것을 볼 수 있습니다. 이러한 @AssistedInejct, @Assisted 뷰모델은 @AssistedFactory를 사용하여 뷰모델의 @Assisted 생성자 파라미터를 주입해 줄 수 있습니다.
```Kotlin
@AndroidEntryPoint
class PlantDetailFragment : Fragment() {

    private val args: PlantDetailFragmentArgs by navArgs()

    @Inject
    lateinit var plantDetailViewModelFactory: PlantDetailViewModelFactory

    private val plantDetailViewModel: PlantDetailViewModel by viewModels {
        PlantDetailViewModel.provideFactory(plantDetailViewModelFactory, args.plantId)
    }
```
```Kotlin
class PlantDetailViewModel @AssistedInject constructor(
    plantRepository: PlantRepository,
    private val gardenPlantingRepository: GardenPlantingRepository,
    @Assisted private val plantId: String
) : ViewModel() {

    val isPlanted = gardenPlantingRepository.isPlanted(plantId).asLiveData()
    val plant = plantRepository.getPlant(plantId).asLiveData()

    fun addPlantToGarden() {
        viewModelScope.launch {
            gardenPlantingRepository.createGardenPlanting(plantId)
        }
    }

    fun hasValidUnsplashKey() = (BuildConfig.UNSPLASH_ACCESS_KEY != "null")

    companion object {
        fun provideFactory(
            assistedFactory: PlantDetailViewModelFactory,
            plantId: String
        ): ViewModelProvider.Factory = object : ViewModelProvider.Factory {
            @Suppress("UNCHECKED_CAST")
            override fun <T : ViewModel?> create(modelClass: Class<T>): T {
                return assistedFactory.create(plantId) as T
            }
        }
    }
}

@AssistedFactory
interface PlantDetailViewModelFactory {
    fun create(plantId: String): PlantDetailViewModel
}
```
## Hilt + Repository + @Module + @InstallIn + @Provide + Component 계층 및 Scope
![image](https://user-images.githubusercontent.com/48902047/149500452-1613ba9d-b391-4925-9bcd-6b5e0e0f045a.png)

 Repository Hilt 를 살펴보겠습니다.

@Singleton 으로 설정하여 어디서나 동일한 객체를 제공하도록 해주게 만들었다. 그리고 매개변수 생성자 객체는 @Inject로 의존성 주입받게 되어있습니다.

아래의 사진은 싱글톤인경우와 싱글톤이 아닌경우의 사진입니다.

<img src = "https://user-images.githubusercontent.com/48902047/149500655-6c3ab855-4d96-479c-8617-6547f8dd1685.png" width="40%" height="40%"> <img src = "https://user-images.githubusercontent.com/48902047/149500795-774bbdfc-c4f8-43a7-bcb6-2fb1d781bef4.png" width="40%" height="40%">

### Repository는 왜 싱글턴 객체여야 하는가?
Repository는 네트워크 작업 혹은 데이터베이스 작업을 위해 만들어진 뷰와 뷰 모델과는 별개의 공간입니다. 만약 싱글턴이 아닌 단순 클래스라고 가정하면, 매번 네트워크 작업 혹은 데이터베이스 작업이 일어날 시 새로운 클래스 객체를 생성한다는 것은 매우 비효율적입니다. 만약 클래스 생성이 오래 걸린다고 가정하면, 네트워크 작업 및 데이터베이스 작업을 하기 위해 클래스 객체를 생성하는 것은 네트워크 처리, 데이터베이스 처리 시간에 더해져 매우 오래 걸릴 것입니다. 그래서 싱글턴 객체로 선언을 하여 항상 어디서든 준비되어 있도록 합니다.
```Kotlin
@Singleton
class GardenPlantingRepository @Inject constructor(
    private val gardenPlantingDao: GardenPlantingDao
) {
```
```Kotlin
package com.google.samples.apps.sunflower.data

import javax.inject.Inject
import javax.inject.Singleton

/*@Singleton으로 어디서나 동일한 객체를 제공합니다*/
/**
 * Repository module for handling data operations.
 *
 * Collecting from the Flows in [PlantDao] is main-safe.  Room supports Coroutines and moves the
 * query execution off of the main thread.
 */
@Singleton
class PlantRepository @Inject constructor(private val plantDao: PlantDao) {
```
UnsplashRepository는 클래스 위에 @Singleton이 되어있지 않습니다. 이 Repository는 꽃들의 정보를 서버 API에서 가져오는 기능을 갖고 있습니다. 그리고 UnSplashService 인터페이스는 NetworkModule에서 @Singleton으로 생성하게 되어있습니다.
```Kotlin
import kotlinx.coroutines.flow.Flow
import javax.inject.Inject

class UnsplashRepository @Inject constructor(private val service: UnsplashService) {
```
```Kotlin
/**
 * Used to connect to the Unsplash API to fetch photos
 */
interface UnsplashService {

    @GET("search/photos")
    suspend fun searchPhotos(
        @Query("query") query: String,
        @Query("page") page: Int,
        @Query("per_page") perPage: Int,
        @Query("client_id") clientId: String = BuildConfig.UNSPLASH_ACCESS_KEY
    ): UnsplashSearchResponse
```
```Kotlin
@InstallIn(SingletonComponent::class)
@Module
class NetworkModule {

    @Singleton
    @Provides
    fun provideUnsplashService(): UnsplashService {
        return UnsplashService.create()
    }
}
```
## DI 패키지의 Module
DI 패키지의 Module 들에 대해 살펴보겠다.  @Module 이 달린 모듈 클래스 들이 모여있습니다.

![image](https://user-images.githubusercontent.com/48902047/149505037-22675562-7da3-466b-b625-9e02ec448960.png)

![image](https://user-images.githubusercontent.com/48902047/149504998-1cb18aa6-28b0-40d6-936e-315e5636aea3.png)

모듈의 설명과 함께  드로이드 나이츠 2020의 찰스개발자님의 그림 자료만 먼저 봐도 이해가 훨씬 수월합니다.
![image](https://user-images.githubusercontent.com/48902047/149505156-67f2a861-5801-4ac2-a628-4defeabed341.png)
![image](https://user-images.githubusercontent.com/48902047/149505291-2ba34166-9624-45c6-bc65-b897c9704a15.png)
![image](https://user-images.githubusercontent.com/48902047/149505302-a6a0a35e-a71e-4181-97f9-6d90e0592040.png)
![image](https://user-images.githubusercontent.com/48902047/149505721-73ed3091-1577-41db-bca3-966e4ee2458c.png)
![image](https://user-images.githubusercontent.com/48902047/149505732-719e65e0-107e-4976-9a6f-8b4dd11d40d7.png)

이제 Sunflower 코드를 살펴보도록 합시다.

먼저 DatabaseModule 입니다. Hilt Module 로 @Module 어노테이션이 붙어있습니다.

```Kotlin
@InstallIn(SingletonComponent::class)
@Module
class DatabaseModule {

    @Singleton
    @Provides
    fun provideAppDatabase(@ApplicationContext context: Context): AppDatabase {
        return AppDatabase.getInstance(context)
    }

    @Provides
    fun providePlantDao(appDatabase: AppDatabase): PlantDao {
        return appDatabase.plantDao()
    }

    @Provides
    fun provideGardenPlantingDao(appDatabase: AppDatabase): GardenPlantingDao {
        return appDatabase.gardenPlantingDao()
    }
}
```
```Kotlin
/** A Hilt component for singleton bindings. */
@Singleton
@DefineComponent
public interface SingletonComponent {}
```
@InstallIn 어노테이션을 지정하여 각 모듈을 사용하거나 설치할 Android 클래스를 Hilt에 알려줍니다. (= Hilt가 생성하는 DI컨테이너에 어떤 모듈을 사용할지 가리킵니다.)

SingletonComponent 라는 Hilt에서 제공하는 기본 컴포넌트이다.

이러한 Hilt Component  세트는 밑 사진의 링크에서 볼 수 있다. 참고로 현재시간 기준 이 컴포넌트는 업데이트 후에 생긴거라 아직 한글 문서에는 나와있지 않다. SingletonComponent 외 다양한 컴포넌트가 있고 세세한 생명주기도 설정할 수 있다. 문서에 들어가서 쭉 한번 읽어봐야한다.

SingletonComponent 로 하는 이유를 생각해보면 DB, 서버API 통신 객체는 여러 하나의 클래스에 종속되어 있는게 아닌 여러 Repository에서 사용할 수 있고, 어디서든 접근이 가능해야 하므로 Singleton 컴포넌트로 작성을 합니다. 그러한 이유로  DatabaseModule과 뒤에 이어서 볼 NetworkModule 은 InstallIn(SingletonComponent::class)를 통해 싱글턴 모듈임을 나타내도록 한다.

Application 에서 주입되고 앱이 시작부터 죽을때까지 말 그대로 Singleton 으로 처음 생성된 같은 객체가 주입될 것이다.

@Porivides 부분을 보겠습니다.

![image](https://user-images.githubusercontent.com/48902047/149507860-9aa8ee98-9129-423f-80f8-275437d4705c.png)

위와 같은 이유로 DatabaseModule 에서 @Provides 어노테이션을 사용하여 Room 데이터베이스 종속 클래스들의 인스턴스를 삽입합니다.

두번째 모듈인 NetworkModule 입니다. 서버와 통신할 API Interface를 UnSplashService의 compnaion object 싱글톤으로 되어있는 Retrofit2 빌더패턴으로 생성해주는 방식으로 객체를 주입시켜줍니다. 나머지 어노테이션은 DatabaseModule과 동일합니다.

```Kotlin
@InstallIn(SingletonComponent::class)
@Module
class NetworkModule {

    @Singleton
    @Provides
    fun provideUnsplashService(): UnsplashService {
        return UnsplashService.create()
    }
}
```
```Kotlin
interface UnsplashService {

    @GET("search/photos")
    suspend fun searchPhotos(
        @Query("query") query: String,
        @Query("page") page: Int,
        @Query("per_page") perPage: Int,
        @Query("client_id") clientId: String = BuildConfig.UNSPLASH_ACCESS_KEY
    ): UnsplashSearchResponse

    companion object {
        private const val BASE_URL = "https://api.unsplash.com/"

        fun create(): UnsplashService {
            val logger = HttpLoggingInterceptor().apply { level = Level.BASIC }

            val client = OkHttpClient.Builder()
                .addInterceptor(logger)
                .build()

            return Retrofit.Builder()
                .baseUrl(BASE_URL)
                .client(client)
                .addConverterFactory(GsonConverterFactory.create())
                .build()
                .create(UnsplashService::class.java)
        }
    }
}
```

