# Project 3 - Hack-A-Snack 
[P3 Homepage](./images/projectthree.png)

## Overview
My third General Assembly was a group project that lasted one week, building a full-stack MERN application.

## Project Brief
You must:

- Work in a team, using git to code collaboratively.
- Build a full-stack application by making your own backend and your own front-end
- Use an Express API to serve your data from a Mongo database
- Consume your API with a separate front-end built with React
- Be a complete product which most likely means multiple relationships and CRUD functionality for at least a couple of models
- Implement thoughtful user stories/wireframes that are significant enough to help you know which features are core MVP and which you can cut
- Have a visually impressive design to kick your portfolio up a notch and have something to wow future clients & employers. ALLOW time for this.
- Be deployed online so it's publicly accessible.
- Have automated tests for at least one RESTful resource on the back-end.
- Improve your employability by demonstrating a good understanding of testing principals.

The necessary deliverables are:

- A working app hosted on the internet
- A link to your hosted working app in the URL section of your Github repo
- A git repository hosted on Github, with a link to your hosted project, and frequent commits dating back to the very beginning of the project
- A readme.md file

## Technologies 

- React
- Express
- React Hooks
- MongoDB
- Mongoose
- JSX
- SASS
- Bulma
- Git and GitHub
- Canva
- Google Fonts
- Insomnia
- Babel
- Axios
- Mocha
- Chai
- Supertest
- React Select
- Speech Synthesis

## Planning
We were keen to have a thorough plan before starting so that we didn't run into problems later on in the project. We spent most of the first day planning the backend controllers and endpoints using flow charts:

[Backend Flow Chart](./images/project3-backend.png) 

We then spent the next day setting up the backend which ran smoothly. 

## Backend
### Building the backend
We set up our backend very quickly - everyone had a hand in building both the backend and the frontend because we thought that would be best as a learning exercise for everyone. We began by configuring our Express app, and setting up Mongoose and connection to our MongoDB Database. We set up our models and seed files collaboratively, and from there we divided the controllers between us in order to save time. 

**Controllers:** myRecipes, recipes, user and reviews. Models: recipeSchema and userSchema. 

I focused on getting the recipes from our API (Edamam), adding user permissions using our secure route middleware, and also recipe creating and editing. We tested our backend as we were creating it using Insomnia to check the endpoints were functioning correctly, and then we all practices writing and running tests using Chai, Mocha and Supertest libraries to make it even more secure, for example:

```
// Test user end points
describe('Testing user end point', () => {
  beforeEach(done => {
    setup(done)
  })
  afterEach(done => {
    tearDown(done)
  })

  // Test 1
  it('Should register a new user', done => {
    api.post('/api/register')
      .send(
        {
          username: 'newuser1',
          email: 'newuser1@newuser1.com',
          password: 'newuser1',
          passwordConfirmation: 'newuser1'
        })
      .end((err, res) => {
        expect(res.status).to.eq(201)
        expect(res.body.username).to.eq('newuser1')
        done()
      })
  })
```

## Frontend
After completing the backend, I created a wireframe for our frontend, which I'll link to [here](https://docs.google.com/presentation/d/1MEEfxSKmHgncob16UPyGYgVdcOPxekc_4wL9MvIu5gw/edit?usp=sharing), so that we would have a clear image of our app before dividing up the components. We broke our app down into several different frontend components in order to keep the code concise. We created components for: all recipes, single recipe page, login, logout and my account page, as well as modals to edit user profile and add/update recipes.

We tried to divide the workload up as evenly as possible, and I focused mainly on the single recipe page as it was one of the larger components with several different moving parts. 

### Single Recipe Page

[Single recipe page](./images/srp.png) 

This involved comment, a star rating system, a save function and an accessibility feature - a recipe reader (as well as buttons to delete and edit the recipe). In keeping with the rest of the app we styled the frontend using **bulma**. The main single recipe page features the star rating system, the fetched recipe ingredients, the edit recipe modal and also the speech synthesis button, which Jess implemented. 

Here is the function used to implement the star rating system, which is submitted alongside the posted review: 

```
  function averageRating() {
    // Access the review array, filter to get an array of ratings excluding falsy value
    // Map over the filtered array to get an array of valid ratings
    const ratingsArray = recipe.review.filter(recipe => recipe.rating).map(recipe => recipe.rating)
    console.log(ratingsArray)
    // Calculate the sum of all ratings
    const ratingsSum = ratingsArray.reduce((acc, rating) => acc + rating, 0)
    console.log(ratingsSum)
    // Calculate the average rating
    const average = ratingsSum / ratingsArray.length
    console.log(average)
    // Round the average rating to nearest 0.5
    return Math.round(average * 2) / 2
  }
```
This is implemented into the JSX alongside the ingredients list and recipe reader like so:

```
            <div className="block box">
              <h5 className="subtitle">{recipe.review.length <= 1 ? `${recipe.review.length} review` : `${recipe.review.length} reviews`}</h5>
              <h5 className="subtitle">Average Rating: </h5>
              <Rating
                start={0}
                stop={5}
                initialRating={averageRating()}
                readonly={true}
                fractions={2}
              />
            </div>
            <h5 className="subtitle">{`Cooking time: ${recipe.cookingTime} minutes`}</h5>
            <h5 className="subtitle">{`Allergens: ${recipe.allergens}`}</h5>
            <div className="box" style={{ maxHeight: '475px', overflow: 'scroll' }}>
              <div className="buttons has-addons is-right">
                <button className="button is-dark is-rounded" onClick={() => speak({ text: ingredientsList })}>
                  Serenade me with the recipe</button>
                <button className="button is-light is-rounded" onClick={cancel}>Stop</button>
              </div>

              <h5 className="subtitle">{'Ingredients: '}</h5>
              {ingredientsList.map((ingredient, index) => {
                return <p key={index}>{ingredient}</p>
              })
              }
            </div>
            <div className="box">
              <h5 className="title">{'Method URL: '}</h5>
              <a className="subtitle" href={recipe.linkOrMethod} target="_blank" rel="noreferrer">{recipe.linkOrMethod}</a>
            </div>
          </div>
        </div>
      </div>
    </div>
  </main >
```

#### Post Review
To prevent the code becoming too long, I split the post review section into another component and imported it into the SRP component. This includes the save recipe function and the comment/review section. For save recipe I created an async function which, when toggled, adds the selected recipe to the user's saved recipe array (so that it appears on their account page): 

```
  async function handleSaveRecipe(recipeId, authToken, event) {
    setToggle(event.target.checked)
    try {
      if (toggle === false) {
        await axios.put(`/api/myrecipes/${recipeId}`, {}, {
          headers: { Authorization: `Bearer ${authToken}` }
        })
        fetchRecipe()
      } else {
        await axios.put(`/api/myrecipes/unstar/${recipeId}`, {}, {
          headers: { Authorization: `Bearer ${authToken}` }
        })
        fetchRecipe()
      }

    } catch (err) {
      console.log(err)
    }
  }
```
For post review, I used form fields in the JSX, and two async functions that ensure the user is logged in by matching the bearer token in order to handle and delete the posted review. 

```
          <div className="field">
            <div className="control">
              <label className="label">Write your Review</label>
              <textarea
                className="textarea"
                placeholder="Write a review..."
                onChange={(event) => setText(event.target.value)}
                value={text}
              >
                {text}
              </textarea>
            </div>
          </div>
          <div className="field">
            <p className="control">
              <button
                onClick={handleReview}
                className="button is-dark">
                Submit</button>
            </p>
          </div>
```



## Challenges 
For me the most challenging aspect of this app was implementing the reviews and making that work alongside the star rating system. This ties into the difficulties I encountered with setting up the single recipe page - working out how to split the components, and how to make everything fit correctly. I think looking back, I should have been a bit more ambitious with my features - but for this project I was focused more on cementing the fundamentals!

I also found the whole process of managing Git and merge conflicts quite hard to wrap my head around, but we treated it very methodically and repeated the process together everyday, which definitely helped.

## Wins
The process of building a group app really helped me to solidify my understanding of Git and working in a team (the idea of working on separate elements during the day, then combining them to creat a single functioning app). It also really helped my understanding of the MVC app structure by seeing how the front end interacts with the backend - for this reason I'm glad we all decided to do bits of everything, rather than allowing some people to focus purely on front end and others on the backend.


## Stretch Goals

- Allow users to reset their password
- Email confirmation 
- User followers 
- Instant messaging system 
- Recipe feed based on followers 
