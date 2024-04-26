//API demo 1 - mainactivity 
package com.french.apidemo

import android.annotation.SuppressLint
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxHeight
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.width
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material3.Checkbox
import androidx.compose.material3.Divider
import androidx.compose.material3.ExperimentalMaterial3Api
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Scaffold
import androidx.compose.material3.Surface
import androidx.compose.material3.Text
import androidx.compose.material3.TopAppBar
import androidx.compose.runtime.Composable
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.style.TextOverflow
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import com.french.apidemo.ui.theme.APIDemoTheme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {

        val vm = TodoViewModel()

        super.onCreate(savedInstanceState)
        setContent {
            APIDemoTheme {
                // A surface container using the 'background' color from the theme
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    TodoView(vm)
                }
            }
        }
    }
}

@SuppressLint("UnusedMaterial3ScaffoldPaddingParameter")
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun TodoView(vm: TodoViewModel) {

    LaunchedEffect(key1 = Unit, block = {
        vm.getTodoList()
    })
    
    Scaffold(
        topBar = {
             TopAppBar(title = {
                 Row {
                    Text(text= "Todos")
             } })
        },
        content = { 
            if (vm.errorMessage.isEmpty()) {
                
                Column(modifier = Modifier.padding(16.dp)) {
                    
                    LazyColumn(modifier=Modifier.fillMaxHeight()) {
                        
                        items(items = vm.todoList) {todo ->
                            
                            Column {
                                
                                Row(
                                    modifier = Modifier
                                        .fillMaxWidth()
                                        .padding(16.dp),
                                    horizontalArrangement = Arrangement.SpaceBetween
                                ){
                                    
                                    Box(
                                        modifier = Modifier
                                            .fillMaxWidth()
                                            .padding(0.dp, 0.dp, 16.dp, 0.dp)
                                    ){
                                        
                                        Text(text = todo.title, maxLines = 1, overflow = TextOverflow.Ellipsis)
                                        
                                    }//Box
                                    
                                    Spacer(modifier = Modifier.width(16.dp))
                                    
                                    Checkbox(checked = todo.completed, onCheckedChange = null)
                                    
                                }//Row
                                
                                Divider()
                                
                            }//Column
                            
                        }//items
                        
                    }//LazyColumn
                    
                }//Column
                
            } else {
                Text(vm.errorMessage)
            }
        }) //content and scaffold
    
}//TodoView

@Composable
fun Greeting(name: String, modifier: Modifier = Modifier) {
    Text(
        text = "Hello $name!",
        modifier = modifier
    )
}

@Preview(showBackground = true)
@Composable
fun GreetingPreview() {
    APIDemoTheme {
        Greeting("Android")
    }
}

//TodoViewModel 

package com.french.apidemo

import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateListOf
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.setValue
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import kotlinx.coroutines.launch

class TodoViewModel: ViewModel() {

    private val _todoList = mutableStateListOf<Todo>()
    val todoList: List<Todo>
        get() = _todoList

    var errorMessage: String by mutableStateOf("")

    fun getTodoList() {

        viewModelScope.launch {
            val apiService = APIService.getInstance()

            try {

                _todoList.clear()
                _todoList.addAll(apiService.getTodos())

            } catch (e: Exception) {
                errorMessage = e.message.toString()
            }
        }

    }//getTodoList


}//ViewModel

//toDo.kt 
package com.french.apidemo

import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory
import retrofit2.http.GET

data class Todo(
    var userId: Int,
    var id: Int,
    var title: String,
    var completed: Boolean
)

const val BASE_URL = "https://jsonplaceholder.typicode.com/"

interface APIService {

    @GET("todos")
    suspend fun getTodos(): List<Todo>

    companion object {

        var apiService: APIService? = null

        fun getInstance(): APIService {

            if (apiService == null) {

                apiService = Retrofit.Builder()
                    .baseUrl(BASE_URL)
                    .addConverterFactory(GsonConverterFactory.create())
                    .build()
                    .create(APIService::class.java)
            }

            return apiService!!

        }//getInstance

    }//companion object

}//interface

//Projct Gradle 
// Top-level build file where you can add configuration options common to all sub-projects/modules.
plugins {
    id("com.android.application") version "8.2.2" apply false
    id("org.jetbrains.kotlin.android") version "1.9.0" apply false
}

//Model App Gradle 
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
}

android {
    namespace = "com.french.apidemo"
    compileSdk = 34

    defaultConfig {
        applicationId = "com.french.apidemo"
        minSdk = 21
        targetSdk = 34
        versionCode = 1
        versionName = "1.0"

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
        vectorDrawables {
            useSupportLibrary = true
        }
    }

    buildTypes {
        release {
            isMinifyEnabled = false
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_1_8
        targetCompatibility = JavaVersion.VERSION_1_8
    }
    kotlinOptions {
        jvmTarget = "1.8"
    }
    buildFeatures {
        compose = true
    }
    composeOptions {
        kotlinCompilerExtensionVersion = "1.5.1"
    }
    packaging {
        resources {
            excludes += "/META-INF/{AL2.0,LGPL2.1}"
        }
    }
}

dependencies {

    implementation("com.squareup.retrofit2:retrofit:2.11.0")
    implementation("com.squareup.retrofit2:converter-gson:2.9.0")

    implementation("androidx.core:core-ktx:1.12.0")
    implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.7.0")
    implementation("androidx.activity:activity-compose:1.8.2")
    implementation(platform("androidx.compose:compose-bom:2023.08.00"))
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.ui:ui-graphics")
    implementation("androidx.compose.ui:ui-tooling-preview")
    implementation("androidx.compose.material3:material3")
    testImplementation("junit:junit:4.13.2")
    androidTestImplementation("androidx.test.ext:junit:1.1.5")
    androidTestImplementation("androidx.test.espresso:espresso-core:3.5.1")
    androidTestImplementation(platform("androidx.compose:compose-bom:2023.08.00"))
    androidTestImplementation("androidx.compose.ui:ui-test-junit4")
    debugImplementation("androidx.compose.ui:ui-tooling")
    debugImplementation("androidx.compose.ui:ui-test-manifest")
}

//API Demo 2
package com.french.apidemo2

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.tooling.preview.Preview
import com.french.apidemo2.ui.HomeScreen
import com.french.apidemo2.ui.theme.ApiDemo2Theme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContent {
            ApiDemo2Theme {
                // A surface container using the 'background' color from the theme
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    HomeScreen()
                }
            }
        }
    }
}

@Composable
fun Greeting(name: String, modifier: Modifier = Modifier) {
    Text(
        text = "Hello $name!",
        modifier = modifier
    )
}

@Preview(showBackground = true)
@Composable
fun GreetingPreview() {
    ApiDemo2Theme {
        Greeting("Android")
    }
}

// folder -- data -- api -- model 
//charater.kt 
package com.french.apidemo2.data.api.model


import com.google.gson.annotations.SerializedName

data class Character(
    @SerializedName("actor")
    val actor: String,
    @SerializedName("alive")
    val alive: Boolean,
    @SerializedName("alternate_actors")
    val alternateActors: List<Any>,
    @SerializedName("alternate_names")
    val alternateNames: List<Any>,
    @SerializedName("ancestry")
    val ancestry: String,
    @SerializedName("dateOfBirth")
    val dateOfBirth: String,
    @SerializedName("eyeColour")
    val eyeColour: String,
    @SerializedName("gender")
    val gender: String,
    @SerializedName("hairColour")
    val hairColour: String,
    @SerializedName("hogwartsStaff")
    val hogwartsStaff: Boolean,
    @SerializedName("hogwartsStudent")
    val hogwartsStudent: Boolean,
    @SerializedName("house")
    val house: String,
    @SerializedName("image")
    val image: String,
    @SerializedName("name")
    val name: String,
    @SerializedName("patronus")
    val patronus: String,
    @SerializedName("species")
    val species: String,
    @SerializedName("wand")
    val wand: Wand,
    @SerializedName("wizard")
    val wizard: Boolean,
    @SerializedName("yearOfBirth")
    val yearOfBirth: String
)

//characterViewModel 
package com.french.apidemo2.data.api.model

import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateListOf
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.setValue
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.french.apidemo2.data.api.APIService
import kotlinx.coroutines.launch

class CharacterViewModel: ViewModel() {

    private val _characterList = mutableStateListOf<Character>()
    val characterList: List<Character>
        get() = _characterList

    var errorMessage: String by mutableStateOf("")

    var loading by mutableStateOf(false)

    fun getCharacters() {

        viewModelScope.launch {
            val apiService = APIService.getInstance()

            try {

                loading = true
                _characterList.clear()
                _characterList.addAll(apiService.getCharacters())
                loading = false

            } catch (e: Exception ) {
                errorMessage = e.message.toString()
                loading = false
            }

        }//launch

    }//getCharacters
}//ViewModel

// Basic Naviagtion Mainactivity 
package com.stein.basicnavigation

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.tooling.preview.Preview
import com.stein.basicnavigation.ui.theme.BasicNavigationTheme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            BasicNavigationTheme {
                // A surface container using the 'background' color from the theme
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    Navigation()
                }
            }
        }
    }
}

@Composable
fun Greeting(name: String, modifier: Modifier = Modifier) {
    Text(
        text = "Hello $name!",
        modifier = modifier
    )
}

@Preview(showBackground = true)
@Composable
fun GreetingPreview() {
    BasicNavigationTheme {
        Greeting("Android")
    }
}

//naviagatio.ky 
package com.stein.basicnavigation

import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.Button
import androidx.compose.material3.Text
import androidx.compose.material3.TextField
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import androidx.navigation.NavController
import androidx.navigation.NavType
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import androidx.navigation.compose.rememberNavController
import androidx.navigation.navArgument

@Composable
fun Navigation() {
    val navController = rememberNavController() //navigation controller state
    
    NavHost(navController = navController, startDestination = Screen.MainScreen.route) {
        composable(Screen.MainScreen.route) {
            MainScreen(navController = navController)
        }

        //detail screen with list of arguments
        //arguments are passed as part of the route url
        //use /{...} for each required parameter /{name}/{id}/...
        //use "?name={name} or ?name={name}&id={id}...
        //if required, the app will crash if not passed a value
        //should also do some sort of validation too
        composable(
            Screen.DetailScreen.route + "/(name)",
            arguments = listOf(
                //need a navArgument for each argument, in our case only 1
                navArgument(name = "name") {
                    type = NavType.StringType //defaults to string
                    defaultValue = ""
                    nullable = true
                }
            )
        ) {entry ->
            //can get entry of each type for each argument, it's a bundle
            DetailScreen(name = entry.arguments?.getString("name"))
        }
    }
}//Navigation

@Composable
fun MainScreen(navController: NavController) {
    var text by remember {
        mutableStateOf("")
    }

    Column (
        modifier = Modifier
            .fillMaxSize()
            .padding(horizontal = 50.dp),
        verticalArrangement = Arrangement.Center
    ) {
        TextField(
            value = text,
            onValueChange = {
                text = it
            },
            modifier = Modifier.fillMaxWidth()
        )

        Spacer(modifier = Modifier.height(8.dp))
        
        Button(
            onClick = {
                //navigate to detail screen
                navController.navigate(Screen.DetailScreen.withArgs(text)) {
//                    popUpTo(Screen.MainScreen.route) {
//                        inclusive = true //so we only have 1 main screen on the stack at a time
//                    }
                }

            },
            modifier = Modifier.align(Alignment.End)
        ) {
            Text(text = "To Detail Screen")
        }//Button
    }//Column
}//MainScreen

@Composable
fun DetailScreen(name: String?){
    //takes the name from the navigation argument
    Box(
        contentAlignment = Alignment.Center,
        modifier = Modifier.fillMaxSize()
    ) {
        Text(text = "$name")
    }
}

//Favorite Feeds
package com.french.favoritefeeds

import android.annotation.SuppressLint
import android.app.AlertDialog
import android.content.Intent
import android.net.Uri
import android.os.Bundle
import android.util.Log
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.background
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.IntrinsicSize
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxHeight
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.width
import androidx.compose.foundation.layout.wrapContentHeight
import androidx.compose.foundation.layout.wrapContentWidth
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.itemsIndexed
import androidx.compose.foundation.lazy.rememberLazyListState
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.foundation.text.KeyboardOptions
import androidx.compose.material3.Button
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import androidx.compose.material3.Text
import androidx.compose.material3.TextField
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateListOf
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.rememberCoroutineScope
import androidx.compose.runtime.setValue
import androidx.compose.runtime.snapshots.SnapshotStateList
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.platform.LocalFocusManager
import androidx.compose.ui.res.stringResource
import androidx.compose.ui.text.TextStyle
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.text.input.ImeAction
import androidx.compose.ui.text.input.KeyboardType
import androidx.compose.ui.text.style.TextAlign
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import androidx.core.content.ContextCompat.startActivity
import com.french.favoritefeeds.ui.theme.FavoriteFeedsTheme
import com.french.favoritefeeds.ui.theme.lightOrange
import kotlinx.coroutines.launch
import kotlinx.coroutines.runBlocking

class MainActivity : ComponentActivity() {
    @SuppressLint("UnrememberedMutableState")
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {

            val context = LocalContext.current

            //instantiate our FavoriteFeedsPreferences class
            val dataStore = FavoriteFeedsPreferences(context)

            var listItems: SnapshotStateList<FeedData> =
                runBlocking {
                    val tempList = dataStore.readAllFeeds()
                    return@runBlocking tempList
                }

//            val listItems = mutableStateListOf(
//                FeedData(path = "cnn_tech.rss", tag = "Tech"),
//                FeedData(path = "cnn_topstories.rss", tag = "Top"),
//                FeedData(path = "cnn_world.rss", tag = "World"),
//            )

            FavoriteFeedsTheme {
                // A surface container using the 'background' color from the theme
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    Feeds(listItems = listItems)
                }
            }//theme
        }//setContent
    }//onCreate
}//Activity

@SuppressLint("UnrememberedMutableState")
@Composable
fun Feeds(listItems: SnapshotStateList<FeedData>) {

    var feedPathState by remember {
        mutableStateOf("")
    }

    var tagState by remember {
        mutableStateOf("")
    }

    val context = LocalContext.current
    val scope = rememberCoroutineScope()
    val dataStore = FavoriteFeedsPreferences(context)

    listItems.sortWith(compareBy(String.CASE_INSENSITIVE_ORDER, { it.tag }))

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(10.dp)
    ) {

        TextField(
            value = feedPathState,
            onValueChange = {
                feedPathState = it
            },
            label = {
                Text(text = "Feed Path")
            },
            maxLines = 1,
            textStyle = TextStyle(
                color = MaterialTheme.colorScheme.primary,
                fontSize = 18.sp
            ),
            placeholder = {
                Text( text = stringResource(id = R.string.queryPrompt))
            },
            keyboardOptions = KeyboardOptions(
                keyboardType = KeyboardType.Text,
                imeAction = ImeAction.Next
            ),
            modifier = Modifier.fillMaxWidth()

        )//textfield

        Spacer(modifier = Modifier.height(8.dp))

        TagRow(
            tag = tagState,
            updateTag = { newTag ->
                tagState = newTag
            },
            feedPath = feedPathState,
            updateFeedPath = { newPath ->
                feedPathState = newPath
            },
            listItems = listItems
        )

        Spacer(modifier = Modifier.height(8.dp))

        //using weight so the next column will fill up remaining space
        Column(
            modifier = Modifier.weight(1f)
        ) {

            Box(
                modifier = Modifier
                    .background(lightOrange)
                    .padding(8.dp)
            ) {

                FeedsRow(listItems = listItems, feedPath = feedPathState,
                    feedTag = tagState,
                    updateFeedPath = { newPath ->
                        feedPathState = newPath
                    },
                    updateTag = { newTag ->
                        tagState = newTag
                    }
                )
            }//Box

        }//Column

        Spacer(modifier = Modifier.height(8.dp))

        Button(
            onClick = {
                listItems.removeAll(listItems)
                scope.launch {
                    dataStore.removeAllFeeds()
                }
            },
            shape = RoundedCornerShape(4.dp)
        )
        {
            Text(
                text = stringResource(id = R.string.clearTags),
                fontSize = 18.sp,
                fontWeight = FontWeight.Bold,
                modifier = Modifier
                    .fillMaxWidth()
                    .wrapContentWidth(Alignment.CenterHorizontally)
            )

        }//Button

    }//Column


}//feeds

data class FeedData(val path: String, val tag: String) //for each row in UI and datastore

@Composable
fun FeedsRow(listItems: SnapshotStateList<FeedData>,feedPath: String, feedTag: String,
             updateFeedPath: (String) -> Unit, updateTag: (String)->Unit) {

    val listState = rememberLazyListState()

    //do need to hoist the following: placeholders for now, will get from datastore
//    val listItems = mutableListOf(
//        FeedData(path = "cnn_tech.rss", tag = "Tech"),
//        FeedData(path = "cnn_topstories.rss", tag = "Top"),
//        FeedData(path = "cnn_world.rss", tag = "Wold"),
//    )

    LazyColumn(
        verticalArrangement = Arrangement.spacedBy(8.dp),
        modifier = Modifier.fillMaxSize(),
        state = listState
    ) {itemsIndexed(
        items = listItems,
        key = { _, listItem -> 
            listItem.hashCode()
        }
        ) { index, item ->

            FeedItemRow(feedItem = item, updateFeedPath, updateTag)
        } //itemsIndex

        //empty list view
        item{
            if (listItems.count() == 0 ) {
                Column(
                    modifier = Modifier.padding(all = 8.dp)
                ) {

                    Text(
                        text = "No Feeds To Display",
                        modifier = Modifier.padding(all = 4.dp),
                        color = MaterialTheme.colorScheme.primary,
                        textAlign = TextAlign.Center,
                        style = MaterialTheme.typography.headlineSmall
                    )

                }//Column
            }//count == 0
        }//item

    }//Column


}//Feeds Row

@Composable
fun FeedItemRow(feedItem: FeedData,
                updateFeedPath: (String) -> Unit, updateTag: (String)->Unit)
{

    val baseUrl = stringResource(id = R.string.searchURL)
    val context = LocalContext.current
    
    Row(
        modifier = Modifier
            .height(IntrinsicSize.Max),
        verticalAlignment = Alignment.CenterVertically
    ) {

        Button(onClick = {
                //create the URL corresponding to the touched button's query
                val urlString = baseUrl + feedItem.path

                //create an implicit intent to launch a web browser
                val webIntent = Intent(Intent.ACTION_VIEW, Uri.parse(urlString))
                startActivity(context, webIntent, null )
            },
            modifier = Modifier.weight(1f), //so takes up remaining space
            shape = RoundedCornerShape(4.dp)
        ) {

            Text(
                text = feedItem.tag,
                modifier = Modifier
                    .fillMaxHeight()
                    .wrapContentHeight(Alignment.CenterVertically),
                fontSize = 18.sp,
                textAlign = TextAlign.Center,
            )

        } //Button

        Spacer(modifier = Modifier.width(8.dp))

        Button(onClick = {
                updateFeedPath(feedItem.path)
                updateTag(feedItem.tag)
            },
            shape = RoundedCornerShape(4.dp)
        ) {
            Text(
                text = stringResource(id = R.string.edit),
                modifier = Modifier
                    .fillMaxHeight()
                    .wrapContentHeight(Alignment.CenterVertically),
                fontSize = 18.sp,
                fontWeight = FontWeight.Bold,
                textAlign = TextAlign.Center,
            )
        } //Button

    }//Row

}//FeedItem

@Composable
fun TagRow(tag: String, updateTag: (String)->Unit,
           feedPath: String, updateFeedPath: (String)->Unit,
           listItems: SnapshotStateList<FeedData>) {

    val context = LocalContext.current //needed for the dialog
    val focusManager = LocalFocusManager.current //needed for dismissing the keyboard
    val scope = rememberCoroutineScope()
    val dataStore = FavoriteFeedsPreferences(context = context)

    Row (
        //use IntrinsicSize so we can center text on the button below by using
        //fillMaxSize/Width/Height
        modifier = Modifier
            .height(IntrinsicSize.Max),
        verticalAlignment = Alignment.CenterVertically
    ) {
        TextField(
            value = tag,
            onValueChange = {
                updateTag(it)
            },
            label = {
                Text(text = "Feed Tag")
            },
            maxLines = 1,
            textStyle = TextStyle(
                color = MaterialTheme.colorScheme.primary,
                fontSize = 18.sp
            ),
            placeholder = {
                Text( text = stringResource(id = R.string.tagPrompt))
            },
            keyboardOptions = KeyboardOptions(
                keyboardType = KeyboardType.Text,
                imeAction = ImeAction.Next
            ),

        )//textfield

        Spacer(modifier = Modifier.width(8.dp))

        Button(
            onClick = {

                if (tag.isNotEmpty() && feedPath.isNotEmpty() ) {
                    //need the following so we can check for no changes
                    val newFeed = FeedData(path = feedPath, tag = tag)

                    if (!listItems.contains(newFeed)) {
                        listItems.add(newFeed)
                        Log.d("BDF", "element added")
                        scope.launch {
                            dataStore.setFeed(newFeed)
                        }
                        listItems.sortWith(compareBy(String.CASE_INSENSITIVE_ORDER, { it.tag }))
                        Log.d("BDF", listItems.joinToString())
                    }
                    //clear the entry fields
                    updateTag("")
                    updateFeedPath("")

                    //hide the keyboard
                    focusManager.clearFocus()
                } //end if
                else {

                    //display a message to user
                    val builder = AlertDialog.Builder(context)
                    builder.setTitle(R.string.missingTitle) //title bar string

                    //provide an OK button that just dismisses the dialog
                    builder.setPositiveButton(R.string.OK, null)

                    //set the message
                    builder.setMessage(R.string.missingMessage)

                    //create the diaalog from Builder
                    val errorDialog: AlertDialog = builder.create()
                    errorDialog.show() //display it

                }//else error
            },
            shape = RoundedCornerShape(4.dp)
        ) {
            Text(
                text = stringResource(id = R.string.save),
                modifier = Modifier
                    .fillMaxHeight()
                    .wrapContentHeight(Alignment.CenterVertically),
                fontSize = 16.sp,
                fontWeight = FontWeight.Bold,
            )
        }//button
    }//Row
}//TagRow


@SuppressLint("UnrememberedMutableState")
@Preview(showBackground = true)
@Composable
fun FeedsPreview() {
    FavoriteFeedsTheme {
        val listItems = mutableStateListOf(
            FeedData(path = "cnn_tech.rss", tag = "Tech"),
            FeedData(path = "cnn_topstories.rss", tag = "Top"),
            FeedData(path = "cnn_world.rss", tag = "Wold"),
        )
        Feeds(listItems = listItems)
    }
}

//ViewModelDemp 

package com.french.viewmodeldemo

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.tooling.preview.Preview
import com.french.viewmodeldemo.ui.theme.ViewModelDemoTheme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            ViewModelDemoTheme {
                // A surface container using the 'background' color from the theme
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    WellnessScreen()
                }
            }//theme
        }//setContent
    }//onCreate
}//Activity


@Composable
fun Greeting(name: String, modifier: Modifier = Modifier) {
    Text(
        text = "Hello $name!",
        modifier = modifier
    )
}

@Preview(showBackground = true)
@Composable
fun GreetingPreview() {
    ViewModelDemoTheme {
        Greeting("Android")
    }
}







