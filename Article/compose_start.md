Jetpack Compose start tutorial. Referencing:

[A Glimse Into Jetpack Compose By Building an App | by Aldo Surya Ongko | Better Programming](https://betterprogramming.pub/a-glimse-into-jetpack-compose-by-building-an-app-a7869723d4e8)

The core idea of Jetpack Compose is **transferring all xml layout files to kotlin files**. To implement this, Google published **Composable Functions**, which can be used like any layout or views. We can declare a Composable Function in another Composable Function just like putting a ListView in a LinearLayout. The demo below is supposed to be like this:

![[Article/resources/Screenshot_20230408_223849_ComposeTutorial.jpg|200]]  ![[Article/resources/Screenshot_20230408_223852_ComposeTutorial.jpg|200]]

An [OutlinedTextField](https://developer.android.com/jetpack/compose/text) on the top, and a RecyclerView-like List below, with 3 columns. This file is called **HomeScreen.kt** which is a kotlin file instead of a kotlin class, because what we are coding is function but xml and it's corrisponding object.

```kotlin
@Composable  
fun HomeScreen(  
    modifier: Modifier = Modifier,   
){  
    var text by remember{ mutableStateOf("") }  
    Column {  
        OutlinedTextField(  
            modifier = modifier.fillMaxWidth(),  
            value = text,  
            onValueChange = {text = it},  
            label = { Text(text = "Param to pass") },  
        )  
  
        LazyVerticalGrid(  
            modifier = Modifier.padding(16.dp),   
            columns = GridCells.Adaptive(minSize = 96.dp),  
            verticalArrangement = Arrangement.spacedBy(16.dp),  
            horizontalArrangement = Arrangement.spacedBy(16.dp),  
        ){  
            for(i in 1..60){
                item {  
                    ProductCard(name = i.toString())  
                }  
            }  
        }
    }  
}
```

Every composable function is configured by it's param **modifier**, which is just like the various properties in a xml layout file. So we can easily pass the value **from parent to it's child to reuse those functionalities**. There's a trick in the codes above, which is the for loop in a composable function. That means I created 60 ProductCard indexed from 1 to 60, which seems a harder job in xml based coding.

The detail implementation of ProductCard will be talked about before long, but that of LazyVertical Grid is illustrated by Google [here](https://developer.android.com/jetpack/compose/lists).

Now let's turn to the element in the list - ProductCard, **which is a composable function itself.** Look! We create two composable functions, and put one in another, **without xml layout inflating or function overriding**. Such technique is sure to be fluent and neat for coders.

```kotlin
@OptIn(ExperimentalMaterialApi::class)  
@Composable  
fun ProductCard(  
    modifier: Modifier = Modifier,  
    name: String = "" 
){  
    Column(  
        modifier = modifier,  
        horizontalAlignment = Alignment.CenterHorizontally,  
    ) {  
        Card() {  
            Image(
                painter = painterResource(id = R.drawable.ic_launcher_foreground),  
                contentDescription = null,  
                modifier = Modifier  
                    .size(40.dp)  
                    .clip(CircleShape)  
                    .border(1.dp, MaterialTheme.colors.secondary, CircleShape)  
            )  
        }  
        Text(  
            text = name  
        )  
    }  
}
```

The constructor of ProductCard have two params: modifier and name. The former is the common property in every composable function, and the later is the number under the icon like the number 8 below:

![[Article/resources/Pasted image 20230408230220.png]]

Because `Alignment.CenterHorizontally` is still an experimental api, so we annotate it at first. Notice that, the passing by of params in HomeScreen to ProductCard does not include the modifier, so in the case above, the 1st param modifier will be it's defalt value - a newer Modifier:

```kotlin
fun ProductCard(  
    modifier: Modifier = Modifier,  
```

But in my final implementation, you will notice that I have deleted some details for the sake of tutorial.

In our MainActivity, you will have done all the things just by putting HomeScreen in it:

```kotlin
class MainActivity : ComponentActivity() {  
    override fun onCreate(savedInstanceState: Bundle?) {  
        super.onCreate(savedInstanceState)  
        setContent {  
            ComposeTutorialTheme {  
                HomeScreen()
            }  
        }    
    }  
}
```

I will not do that, however, for making a bigger and fancier demo.