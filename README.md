# Notes for Mod 4 Retake
So all the stuff here is fairly subjective. There don't seem to be a ton of hard and fast rules in React, so you can technically structure your program in whatever way you want. But this is how I, personally, have come to understand React. Hope it helps! :)

## Basic Structure of React Apps
Top Level: App

Middle Level: Container

Middle Level: Form & List

Bottom Level: ListItem

```
       App
        |
    Container
     /     \
  Form    List
            |
         ListItem
```

* The bottom levels (e.g. ListItem) are more likely to be functional/presentational components

## The Container
* Considering the structure shown above, the container often acts as a middle man between the form and the list
* Any form submissions need to be communicated to the list; the container state acts as a conduit for the passing of information between the list and the form
* The container state often includes:
  * an array that stores all of the objects in the list
  * if there are buttons (e.g. 'edit' or 'show more' buttons) on the list or if the list items are clickable, then the container might have an object that stores whatever list item the user has currently selected

```
class Container extends React.Component {
  state = {
    listArray = [],
    currentListItem = {}
  }
}
```

## Form
* Form components will likely have their own state, in order to keep track of the changing inputs on the form. This is especially true for text fields.
* You can add a 'name' attribute to your input fields. This will allow you to reuse your onChange function using 'event.target.name':

```
class Form extends React.Component {
  state = {
    name: '',
    age: ''
  }

  // in the function below, the 'event.target.name' will be 'name' for the name text input and 'age' for the age text input

  handleInputChange = (event) => {
    this.setState({ [event.target.name]: event.target.value })
  }

  // NOTE: the form below has a handleSubmit function that is passed down from the parent component; you typically keep track of onChange events in the Form and then pass the state up to the parent upon form submit

  render() {
    <form onSubmit={() => {this.props.handleSubmit(this.state)}}>
      Name: <input onChange={this.handleInputChange} type="text" name="name" value={this.state.name}/>
      <br/>
      Age: <input onChange={this.handleInputChange} type="text" name="age" value={this.state.age}/>
    </form>
  }
}

```

## List
* The list will often have a child component for rendering each item in the list.
* Typically, the list will receive a list array from the parent container. You use this array to populate your list.

```
const List = (props) => {
  return (
      <ul>
        {props.listArray.map((item) => {
          <ListItem itemData={item}/>
        })}
      </ul>
  )
}
```

* NOTE: In the .map function above, we passed the item data as props to each individual list item. It's super helpful for each ListItem to have access to its own data, especially if that data includes a unique ID.

## ListItem
* Ideally, this will just be a presentational component that renders an item for a list.

```
const ListItem = (props) => {
  return (
    <li>Name: {props.itemData.name}, Age: {props.itemData.age}</li>
  )
}
```

## Presentational vs Class Components
* The main differences are that presentational components:
  * do not have state
  * do not have lifecycle methods
* You can technically create functions in presentational components by storing them in variables above your return statement (probably not best practice but it's possible to do):

```
const List = (props) => {

  const handleChange = (event) => {
    console.log(event.target)
  }

  return (
    <ul onChange={handleChange}>
      <ItemList />
    </ul>
  )
}
```

## Misc. Tips & Tricks

### setState
* setState is asynchronous so if you try to run a function (which uses the newly updated state) right after setting state, you will get errors
* The setState function takes a callback as its second argument; you can use this callback to call upon your newly updated state
* It takes both anonymous arrow functions or named functions

```
// This will not work:

this.setState({ currentListItem: 'test' })
console.log(this.state)

=> the console will log the previous state of currentListItem
```

```
// This will work:

this.setState({ currentListItem: 'test'}, () => console.log(this.state))

=> the console will now log 'test' for currentListItem
```

### Closures
* When passing functions down as props, you'll want to take special care when passing arguments to those functions
* Event is passed down automatically; no extra work is needed
* However, when passing any other arguments, you need to do so manually
* If you do not wrap the function in a closure, the function will run once on page load and it will not run again

```
// This will not work:

<form onSubmit={this.props.handleSubmit(this.state)}></form>
```

```
// This will work:

<form onSubmit={() => {this.props.handleSubmit(this.state)}}>
```

### Rerendering Lists
* A couple ideas for editing an item on a list & reflecting that change on the frontend:
  * You will need to manipulate your list array in your container's state; this is probably what you are using to render the list, so if you change the state of the array, it will likely change the list on the frontend
  * You can do this by slicing the array and using the spread operator to create a new array
  * You can also do a 'get' fetch request to retrieve the updated data from your database. You then set the state of your list array to the response.
