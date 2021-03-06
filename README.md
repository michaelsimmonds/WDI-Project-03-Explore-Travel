# General Assembly Project 3 : Full Stack RESTful App React/Express/MongoDB

### Timeframe

5 days in a group of 3 developers

## Technologies used

* Sytling: HTML5 / SCSS / Bulma
* Front-end: React.js / Webpack / Mapbox-GL / React Beautiful DnD /
Axios / Promise / FileStack React
* Server: Node.js / Express / JWT / Request-Promise
* Database: MongoDB / Mongoose / Models with Reference and Embedded
* Integration testing: Mocha / Chai
* DevOps: Git / GitHub / Heroku

## External APIs

* FileStack.js
* Mapbox Auto Complete
* Darksky
* Twitter

## Application - Plan Your Next World Trip

![image](https://user-images.githubusercontent.com/39668354/52906178-84ddcb80-323e-11e9-90e9-3df5fa01bc69.png)

You can find a hosted version here ----> [Heroku](https://project-4-wdi.herokuapp.com/)

### Overview

This application is a website proposing a service for planning your trip around the world. The idea is to propose a list of destination to any user visiting our website. If the user is interested, he can sign-in with us and accessing more information like:
* A personalised dashboard where the user can add his destination and re-order the sequence in which he desires to travel.
* Each destination will also have more information like live forecast, average budget or the latest Tweets talking about the place.

As our destination database can evolve everyday, we also created a feature allowing admin users to add new places and destinations.

![ezgif com-video-to-gif 2](https://user-images.githubusercontent.com/39668354/52919658-99809900-32fc-11e9-80f1-60f7f3031abf.gif)

---

### Development process

#### Communication

Key point for this project and any project, we met every morning in a quiet room, broadcasting the latest version of the application and discussing about today's priority.

We also tend to work in a 'war zone' kind of mode where we could code next to each other. This allowed rapid decision making process when an issue came up or a design concern was raised.

#### Task management

Working in a group of three is a huge advantage as long as each other understand the work that needs to be done. We used Trello quite heavily to create tickets and assigned tasks. We decided to assign one person to build the back-end, another to build the frontend and the third to experiment with new technologies that we wanted to use in our version 2.0 (fully featured product).

#### Branching and Conflict resolution

We use a simplified version of the GitFlow branching system by using three different types of branches:

1. Development: this is the main branch where anyone could create feature branches from and merge their work back in. It was important that none of us ever commited broken code on this branch.

2. <feature-branch>: those branches were created by any member of the group in order to develop new features. The branches needed to be named according to the feature developed (i.e. <login-route>).
  
3. Master: this branch was dedicated to deployment. No commit should never happen in this branch. When a version of the app was ready to be deployed in production (in Heroku), the development branch was merged into master and then pushed ot Heroku.

Conflicts were mitigated by making sure that everyone pulled everyone's changes every morning to avoid long divergent branches. When a conflict occurred we all discussed it to ensure that no information would be lost.

---

### Architecture

Our application is following as much as it can the RESTful paradigm, let's have a look at what is happening when an user is navigating to the index page of the places (destination):

![Screenshot 2019-03-25 at 19 19 27](https://user-images.githubusercontent.com/43914382/54947761-fcd09d00-4f32-11e9-908a-d32ad29b082a.png)

1. An axios request will be sent to our back-end API requesting all the places to be sent back to the front-end:

```
componentDidMount() {
  //If user is logged in, the response will contain only the places
  //that the user doesn't have already in his dashboard
  axios.get('/api/places', {
    headers: Auth.isAuthenticated() ?
      { Authorization: `Bearer ${Auth.getToken()}`} : null
  })
    .then(res => this.setState({ places: res.data }))
}
```

2. Our Express server is using Router to navigate the request toward the right controller.

```
router.route('/places')
  .get((req, res, next) => {
    if(req.headers.authorization) secureRoute(req, res, next)
    else next()
  },placesController.index)
  .post(secureRoute, placesController.create)
  ```

3. The controller will then handle the request and retrieve the data from our database:

```
function indexRoute( req, res ){
  Place
    .find()
    .then(places => places.filter(place => {
      if(req.currentUser) {
        //return false if user already added place to his trip
        return !req.currentUser.places.some(userPlace => userPlace.equals(place._id))
      } else return true
    })
    )
    .then(places => res.status(200).json(places))
}
```

4. The model is created as a blueprint of our collection for the place:

```
const placeSchema = new mongoose.Schema({
  name: { type: String, required: 'Name required' },
  country: { type: String, required: 'Country required' },
  image: { type: String, required: 'Image url required' },
  descriptLong: { type: String, required: 'Long description required' },
  descriptShort: { type: String, required: 'Short description required'},
  geog: { type: Array, required: 'Lat/lng required, in array form: [lat, lng]' },
  budget1: { type: String },
  budget2: { type: String },
  budget3: { type: String },
  comments: [ commentSchema ]
})
```

---

### Challenges

* Free APIs for daily budget in each country was hard to find, we were looking for the BigMac and Cock index type of API to give the user an idea of cost of living.

---

### Wins

1. Team organisation: daily stand-up, daily merging of each others branches and democratic decision making gave our team direction and minimised time spent resolving conflicts arising from divergent paths. We all kept in sync during this time which imporved the productivity and atmosphere in the team.

2. How the page's content alters depending on whether the user is authenticated or not. For example, on the show page, we only show current weather, tweets and comments if the user is logged in.

3. The drawing line on the map was quite a challenge since every point had to be calculated for each increment [X (latitude), Y (longitude)]. The idea was to use the tan(O) = opposite / adjacent trigonometric formula to find, for each time we increment X, the corresponding Y point.  

---

### Future Features / Enhancements

* The "place" model which contains all the information about a single destination in the MongoDB could have a user field. This could be populated with the users who have added that place to their trip so they could potentially be connected together.

* Email validation and password reset.

* Option for the user to add a place of his choice (which might not be part of the main index list that our service proposes).

* More information on the dashboard about the overall trip, total cost estimation, time in weeks, trip lenght in Km...
