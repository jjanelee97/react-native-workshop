## Set Up:

:warning: We should already have node installed on our machines, but just in case, let's go way to the beginning:

`$ brew install node`

🚀 And we should install Watchman, too, since react-native depends on it.

`$ brew install watchman`

Watchman is a file watching service that records when files change, and triggers actions when it detects changes.

🚀 Alright, now we're ready to use react-native! We'll want to install the Command Line Interface so we can call react-native commands from the terminal:

`$ npm install -g react-native-cli`



Great! Now we're ready to create our repo.

```
$ react-native init VidSearch
$ cd VidSearch
$ react-native run-ios
```

Let's take a sec to talk about how the simulator works. There are two things you can do:

`command-D` --> "Enable Hot Reloading"
`command-R` --> refreshes simulator INSTANTLY with your changes

It's pretty groundbreaking.  React Native is cutting through all the overhead for us so we can do instantaneous reloads without all the wait time.



## Installing Dependencies

We're going to need a few dependencies from our trusty friend, the Node Package Manager aka NPM.
🚀 Since we're making calls to the YouTube api, it would help if we made GET calls with axios, so:


`$ npm install --save axios`


🚀 There's one additional component we'll be using in the workshop: [react-native-search-box](https://github.com/crabstudio/react-native-search-box), a simple input field made to look like the classic iOS search bar.

`$ npm install --save react-native-search-box`

# THE ACTUAL CODE

## Basic Navigation

🚀 Create a new directory in the top level of the project folder called `components`.
Then create two new files: `components/search.js` and `components/featured.js`.

```js
import React, { Component } from 'react';

import {
    NavigatorIOS,
    View,
    Text,
    StyleSheet,
  } from 'react-native';

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
});

class Search extends Component {
  render() {
    return (
      <View style={styles.container}>
        <Text style={styles.text}>
          Search component
        </Text>
      </View>
    );
  }
}

module.exports = Search;
```

🚀 And for do the same for your `featured` component.  Refactor as necessary!


🚀 Now let's connect them all together. Go to `index.ios.js` in your file directory.

This is the top-level file for iOS dev in react native.

Add the following code:

```js
import React, { Component } from 'react';
import Featured from './components/featured';
import Search from './components/search';

import {
  AppRegistry,
  TabBarIOS,
} from 'react-native';

class VidSearch extends Component {

  constructor(props) {
    super(props);
    this.state = {
      selectedTab: 'search'
    };
  }

  render() {
    return (
      <TabBarIOS selectedTab={this.state.selectedTab}
        translucent={false}
        unselectedItemTintColor='#9E9E9E'
        tintColor='#c4302b'
      >
        <TabBarIOS.Item
          selected={this.state.selectedTab === 'search'}
          systemIcon='search'
          onPress={() => {
            this.setState({
              selectedTab: 'search'
            });
          }}>
          <Search />
        </TabBarIOS.Item>
        <TabBarIOS.Item
          selected={this.state.selectedTab === 'featured'}
          systemIcon='featured'
          onPress={() => {
            this.setState({
              selectedTab: 'featured'
            });
          }}>
          <Featured />
        </TabBarIOS.Item>
      </TabBarIOS>
    );
  }
}

AppRegistry.registerComponent('VidSearch', () => VidSearch);
```

🚀 Now go to your simulator and press `Cmd+R`. You should have this:

![tab bar](./images/tab-bar.png)




## Adding Content to Featured page

Let's make the featured page look a little less boring. Replace all the code in it with this:

```js
import React, { Component } from 'react';
import ImageView from './imageView';

import {
    StyleSheet,
    View,
    NavigatorIOS,
    Text,
    Image,
    StatusBar,
  } from 'react-native';

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
});

const Featured = (props) => {

  return (
    <View style={styles.container}>
    <StatusBar
     backgroundColor="blue"
     barStyle="light-content"
   />
    <NavigatorIOS
      style={styles.container}
      translucent={false}
      barTintColor='#c4302b'
      titleTextColor='white'
      initialRoute={{
        title: 'Yay React',
        component: ImageView,
      }}
    />
    </View>
  );
};


module.exports = Featured;
```

🚀 Now create a new file called `components/imageView.js`. We'll just put this nice react logo in it, so the page isn't so dreadfully boring.

Here's some more code!

```js
import React from 'react';
import {
  View,
  Image,
} from 'react-native';


const ImageView = (props) => {
  return (
    <View>
      <Image
        style={{ width: 400, height: 300 }}
        source={{ uri: 'https://facebook.github.io/react/img/logo_og.png' }}
      />
    </View>
  );
};

module.exports = ImageView;
```

:snowflake: Now, instead of some gross text up in the top corner, we have this nice react logo:

![featured tab](./images/featured.png)




## Adding Content to the Search Page
So we've got some basic navigation working on the app, but it looks pretty boring. Let's make some cool stuff on the search tab, like how about a nice table view?

🚀 Create a new file: `components/video_list.js`. Add some imports:

```js
import React, { Component } from 'react';
import youtubeSearch from '../youtube-api';
import axios from 'axios';
import Search from 'react-native-search-box';

import {
    StyleSheet,
    View,
    Image,
    Text,
    TextInput,
    ListView,
    TouchableHighlight,
  } from 'react-native';
```

🚀 Now that that's there, let's import it into `search.js` so we can use it.


🚀 And lets create a new class component:

```js
import VideoDetail from './video_detail';

// Add styling here

class VideoList extends Component {

  constructor(props) {
    super(props);
    this.state = {
      query: 'dog',
      isLoading: true,
      dataSource: new ListView.DataSource({
        rowHasChanged: (row1, row2) => row1 !== row2,
      }),
    };
  }

  //---------- componentDidMount here! -----------//

  //------------ put fetchData here! -------------//

  renderLoadingView() {
    return (
      <View style={styles.loading}>
        <Text>
          Loading videos...
        </Text>
      </View>
    );
  }

  //Handle your transition to the detail page
  //pass along the clicked video into props to display it!
  showVideoDetail(video) {
    this.props.navigator.push({
      title: video.snippet.title,
      component: VideoDetail,
      passProps: { video },
    });
  }

  renderVideoCell(video) {
    return (
      <TouchableHighlight onPress={() => { this.showVideoDetail(video); }} underlayColor="#dddddd">
        <View>
          <View style={styles.container}>
            //----- TableView Content should go here -----//
          </View>
          <View style={styles.separator} />
        </View>
      </TouchableHighlight>
    );
  }

  render() {
    if (this.state.isLoading) {
      return this.renderLoadingView();
    }
    return (
      <View style={{ marginBottom: 150 }}>
        <Search
          backgroundColor='#c4302b'
          showsCancelButton={false}
          textFieldBackgroundColor='#c4302b'
          onChangeText={(query) => {
            this.setState({ query });
            // Call fetchData here! take this comment out after
            this.fetchData();
          }
          }
        />

        <ListView
          dataSource={this.state.dataSource}
          renderRow={this.renderVideoCell.bind(this)}
          style={styles.listView}
        />
      </View>
    );
  }
}

module.exports = VideoList;
```

** IMPORTANT: the comments are to guide you, but do NOT leave them in when you want to run your code.  You need to take them out before you run your simulator. **

🚀 And we should add in some styles too. Let's make it a little more interesting this time:

```js
const styles = StyleSheet.create({
  container: {
    flex: 1,
    flexDirection: 'row',
    justifyContent: 'flex-start',
    alignItems: 'flex-start',
    backgroundColor: 'white',
    padding: 10,
  },
  thumbnail: {
    width: 80,
    height: 80,
    marginRight: 10,
  },
  rightContainer: {
    flex: 1,
  },
  title: {
    fontSize: 16,
    fontWeight: 'bold',
    marginBottom: 3,
  },
  subtitle: {
    fontSize: 12,
  },
  separator: {
    height: 1,
    backgroundColor: '#dddddd',
  },
  listView: {
    backgroundColor: 'white',
  },
});
```



🚀 Take a look at the simulator. We've now got some text indicating that the videos are loading.

This is the default text we've provided if the API call hasn't returned videos yet. Since we haven't made an API call yet, that definitely makes sense.

🚀 Time to use that `axios` call to populate the table view!

Add in the following lines:

```js
<Image
  source={{ uri: video.snippet.thumbnails.default.url }}
  style={styles.thumbnail}
/>
<View style={styles.rightContainer}>
  <Text style={styles.title}>{video.snippet.title}</Text>
  <Text style={styles.subtitle}>{video.snippet.description}</Text>
</View>
```

🚀 Hmm...simulator says still just loading videos. That's because we need to actually gather our data from Youtube. Let's add in the fetchData method to make our API call.

```js
fetchData() {
  youtubeSearch(this.state.query)
     .then((responseData) => {
       this.setState({
         dataSource: this.state.dataSource.cloneWithRows(responseData),
         isLoading: false,
       });
     })
     .done();
}
```

Where should we call this from? It would be nice if we could get the data from YouTube as soon as we get to the page.

🚀 Create a function `componentDidMount` in `video_list.js`. Inside it, make a call to `fetchData`.

:snowflake: Now when the page loads, we'll call `fetchData` to populate our list view.

🚀 Alright, let's update the `search.js` file to have a table view that lists all our videos. Replace the return statement in the render function with the following:

```js
<View style={styles.container}>
  <StatusBar
    backgroundColor="blue"
    barStyle="light-content"
  />
  <NavigatorIOS
    style={styles.container}
    translucent={false}
    barTintColor='#c4302b'
    titleTextColor='white'
    tintColor='white'
    initialRoute={{
      title: 'Featured Videos',
      component: VideoList,
    }}
  />
</View>
```

Ah darn, one other thing. We need to actually have a reference to the API, right? This next part should (hopefully) look super familiar.

🚀 Create a new file at the top level of your project, `youtube-api.js`.

Sound familiar?  We did this in short assignment 4, and we will be using the exact same api for this react-native app!  That's coooool.

Go ahead and find that file and copy it here.  We need to do this because we need your individual api key, which you already made in sa4.  

- Accidentally deleted your API key? No biggie. Just follow the [old instructions from sa4](http://cs52.me/assignments/sa/react-videos/#youtube-api).

🚀 What's this videoDetail thing? We'll also need to create that. Make a new file called `compnents/video_detail.js` and paste in this code:

```js
import React, { Component } from 'react';

import {
    StyleSheet,
    Text,
    View,
    Image,
    WebView,
  } from 'react-native';

const styles = StyleSheet.create({
  container: {
    alignItems: 'center',
  },
  image: {
    width: 107,
    height: 165,
    padding: 10,
  },
  description: {
    padding: 10,
    fontSize: 15,
    color: '#656565',
  },
});

class VideoDetail extends Component {
  render() {
    const video = this.props.video;
    const description = video.snippet.description || '';
    const vidId = video.id.videoId;
    return (
      <WebView
          style={styles.frame}
          source={{uri: `https://www.youtube.com/watch?v=${vidId}`}}
          renderLoading={this.renderLoading}
          renderError={this.renderError}
          automaticallyAdjustContentInsets={false}
      />
    );
  }
}

module.exports = VideoDetail;
```

:snowflake: This is a little different from what we've been doing. The WebView component is a sort of hybrid component that's actually just rendering a webpage. The `source` prop holds a uri that's called as if in a browser and then displayed in our application. Notice how it looks just like watching youtube on a mobile device. Pretty cool that we can do this within our application alongside native components, huh?

Here's what the app should be looking like now:
![finished app](./images/finished.png)

🚀 Now that the app is complete, we're using all the styling we pasted in awhile ago. Now it's your turn: play around with the styling in `video_list.js`. If you haven't enabled hot-reloading yet, do that, it'll make it easy to see all your styling changes.

:camera: Make the styling uniquely your own. Then, search for some unique searchterm and take a screenshot. You'll upload this to your repo to turn in.

## And We Are Done!

What we did in just an hour:
- [x] Learned how to set up a new iOS project in react native (from scratch!)
- [x] Implement Tab Bar navigation (without Swift code)
- [x] Learn how to use the react-native simulator
- [x] Made a table view on an iPhone in JavaScript
- [x] Make an API call to YouTube using axios
