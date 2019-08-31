---
layout: post
title:      "Javascript - Tackling The Pokemon Teams Lab"
date:       2019-08-31 11:49:29 -0400
permalink:  javascript_-_tackling_the_pokemon_teams_lab
---


***"Javascript is easy!", said no one ever. ***

### Asynchrony and using .then()

In particular, wrapping my head around the concept of asynchrony proved to be a challenge. There were times where I felt like my code was completely fine but it wasn't being reflected in the output, especially if some part of the process depended on a fetch request. For instance, the pokemon lab involved adding event listeners for certain buttons but before that could happen, we needed to render the html for each trainer and their corresponding "add pokemon" button and "release" buttons through a fetch request. The thing about fetch requests is that they are asynchronous functions and Javascript works by placing them in a task queue first and calling all other synchronous functions before allowing the fetch request to complete. Therefore, if I wanted to add an event listener to a html element that would exist only after the fetch request was complete, I would have to employ the use of `.then()` which sort of ensures that a Promise is returned (basically a response indicating that the fetch was completed successfully) before calling the function. 

What I learned through this process was `.then()` is quite fussy about how it is employed. For instance, I had a function called `addReleaseListeners()` that didn't require an argument. Now if we wanted to call this function after completing the fetch request, we could do it as such: 

```
fetch(TRAINERS_URL)
.then(resp => resp.json())
.then(json => makeDiv(json))
.then(addReleaseListeners)
```

On the other hand, if we had done something like this (including the blank parantheses after the function name): 

```
.then(addReleaseListeners())
```

Javascript would've went ahead and called `addReleaseListeners()` while the fetch was still running in the background, sort of ignoring `.then()`. 

What if we wanted to call a function that requires an argument, e.g. `postAddPokemon(id)`? This would be the right way to do it: 

```
  fetch('http://localhost:3000/pokemons',{
    method: 'POST',
    headers: {
      "Content-Type": "application/json",
      "Accept": "application/json"
    },
    body: JSON.stringify({"trainer_id":`${id}`})
  })
  .then(()=>fetchLastPokemon(id)) 
```

Many hours were spent trying to figure this out, when all that was causing the problem was some syntactic nuance, a painful but nonetheless necessary lesson. 

### Bubbling
Another useful concept I learned in this lab was the principle of bubbling when adding event listeners. Here is a short definition taken from [javascript.info](https://javascript.info/bubbling-and-capturing):

***When an event happens on an element, it first runs the handlers on it, then on its parent, then all the way up on other ancestors.***

What this means is that if we were to click on anything within a certain element (a button within a div for example), the parent element would know about this click event that just happened and from there, we can then specify conditions on what to run depending on which element (i.e. `event.target`) was clicked. In the context of the Pokemon Teams lab, each trainer card lies within the `<main>` element of the DOM, and each "Add Pokemon" button lies within the trainer card div, and each "Release" button lies within the `<li>` elements nested within the `<ul>` within the trainer card div. 

![Pokemon Teams HTML](https://i.imgur.com/VmIgJEo.png)

If we were to click on any of those buttons mentioned, the knowledge of this click event "bubbles" all the way up to the `<main>` element. So how is this all relevant? Well, it means that we wouldn't have to parse the whole html for each individual "Add Pokemon" button or "Release" button add event listeners to them individually. This not only takes up resources and time, but it is also dependent on whether the fetch request has been completed to render all these button elements (think asynchrony and the complications that it brings). To put simply, instead of adding an event listener to each individual button using our code, e.g.:

```
function addBtnListeners(){
  let trainerDivs = document.querySelectorAll('.card')
  trainerDivs.forEach(trainer =>{
    let addBtn = trainer.querySelector('button')
    let id = trainer.getAttribute('data-id')
    let list_size = trainer.getElementsByTagName('li').length
    addBtn.addEventListener('click',function(event){
      if (list_size < 6){
        postAddPokemon(id)
      }
    })
  })
}
```

We could just add an event listener to the `<main>` element which envelopes all the buttons that the user would ever need to click on. 

```
body.addEventListener('click',function(event){
  if(event.target.className=='add'){
    let id = event.target.dataset.trainerId //makes data-trainer-id = '2' equal to {trainerID: '2'}
    let list_size = event.target.nextElementSibling.children.length //to get the ul which comes right after the button
    if (list_size < 6){
      postAddPokemon(id)
    }
  }
})

function postAddPokemon(id){
  fetch('http://localhost:3000/pokemons',{
    method: 'POST',
    headers: {
      "Content-Type": "application/json",
      "Accept": "application/json"
    },
    body: JSON.stringify({"trainer_id":`${id}`})
  })
  .then(()=>fetchLastPokemon(id))
```

In the code above, we are using `event.target` to identify if the element that was just clicked on has a className of "add", only if that holds true would we carry out our intended response from that particular click. We could do the same for the release buttons by specifying the condition ` if(event.target.className=='release')`. 


