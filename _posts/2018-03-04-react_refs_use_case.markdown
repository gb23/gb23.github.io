---
layout: post
title:      "React Refs Use Case"
date:       2018-03-05 03:23:00 +0000
permalink:  react_refs_use_case
---

For my react-redux project, I designed the user experience to center around navigating through different cards (divs) with the arrow keys on the keyboard. Check out the walkthrough of the app on YouTube for a full understanding of the project. [link to be posted].  You'll notice that in the project, when a user navigates to a new card, React re-renders the new content and maintains focus (selection/highlight) on the card to which the user navigated.  The maintained focus on the proper card is accomplished using the concept of Refs in React.  

**Without implementing refs in my project, a user would select a div and when the component re-renders, the selection of the card would go away, causing the user to have to reselect the card. Ugh.  The correct behavior, accomplished with refs: The user selects a card, content is updated, and the card is still selected. Yay!**

## Note: Use Refs Cautiously
Refs are a break from the typical react dataflow of props being the only way that a parent component interacts with it's child component.  That is, passing of new props from parent to child causes the child to re-render with the new prop values. This is *declarative programming* in React.  You aren't concerned with how to get the child component to display the new results; you are simply concerned what the child component will look like with the new data from the props.

Most of the time this is all you need in React.  However, there are a few situations where Refs are kosher and fit a good use case.  One of these cases is managing focus. Using refs follows the *imperative programming* paradigmn: we are now concerned with the "how".  *How* are we going to maintain focus on an element after re-rendering?

## Why Refs Are Needed For Managing Focus

Let's talk through what is happening in plain English.

### Normal React Flow:
1. Updated data can be passed to child component
2. Child takes any data passed to it and presents it to user.  That's it.  It ends at 2.  Note that it's unidirectional.  Parent to child.

### What Somehow Needs to Happen for Focus to be Maintained:
1. Updated data can be passed to child component
2. Child takes any data passed to it and presents it to user.
3. The Child informs the Parent, "hey, after the update of information leading to re-render, I need you to automatically focus me."

How is the parent supposed to know which div is 'me' to automatically focus?  The parent could have many divs.  We need to be less declarative and more imperative here.  We tell the parent which child div is 'me' by including a ref callback in the child div.  This reference is then assigned to a class variable in the Parent .  Now whenever we call this variable of the parent, we know which of the children is 'me'!

Here's a [JS Fiddle](https://jsfiddle.net/69z2wepo/125303/) of the code I will be discussing.  When you visit this link, click on the 'ChildComponent1' that is rendered. Then arrow on the keyboard to the right or left.  You'll see that you switch to 'ChildComponent2'.  If you then press left of right again, you'll be back to 'ChildComponent1'. You can continue pressing the arrow keys at will to switch between the component renderings.  **Hugely important: You do NOT need to re-select the div everytime the ChildComponent switches between `ChildComponent1` and `ChildComponent2`.  This is because of refs!**  Take out the refs and see what happens! It will be like this: click on div; arrow over; click on div; arrow over; click on div; arrow over....


Here is the code I created in the JS Fiddle demo.
```javascript
class ParentContainer extends React.Component {
    componentDidUpdate() {
    	this.divElement.focus();
    	console.log("update!");
    }

    constructor(){
      super();
      this.state = {
        fromFirst: true
      }
      this.divElement = null;
    }
    
   handleDown1 = (event) => {
        if(event.key === "ArrowRight" || event.key === "ArrowLeft"){
      	    this.setState({fromFirst: false})
      	    console.log("Fromfirst", this.state.fromFirst);
      }
    }
  
   handleDown2 = (event) => {
        if(event.key === "ArrowRight" || event.key === "ArrowLeft"){
      	    this.setState({fromFirst: true})
      	    console.log("Fromfirst", this.state.fromFirst);
        }
   }

   divReference = (el) => {
   		this.divElement = el
   }
  
  render() {
    return (
    	<div>
      	    { this.state.fromFirst ?
            < ChildComponent1 divRef={this.divReference}  keyDown={this.handleDown1} /> 
            :
            < ChildComponent2 divRef={this.divReference} keyDown={this.handleDown2} />
            }
   	</div>
    );
  }
}

const ChildComponent1 = (props) => {
    return (
	    <div tabIndex="0" onKeyDown={props.keyDown} ref={props.divRef}>
                ChildComponent1
  	    </div>
    );
}

const ChildComponent2 = (props) => {
    return (
	    <div tabIndex="0" onKeyDown={props.keyDown} ref={props.divRef}>
  	        ChildComponent2
  	    </div>
    );
}

ReactDOM.render(
  <ParentContainer/>,
  document.getElementById('container')
); 
```


We have a `<ParentContainer/>` that renders either a `<ChildComponent1>` or `<ChildComponent2>` based on a boolean state value of the `ParentContainer` named `fromFirst`. This variable answers the question when toggling between `ChildComponent1` and `ChildComponent2`: are we toggling from the first child?   When this state is true, we render `<ChildComponent2>`. (We are coming *from first* and going to second) and when the state is false, we render `<ChildComponent1>`. (We are coming from second and going to first).  The parent checks `fromFirst` in its state and makes the decision which child component to render.  This data flow, again, is parent to child and is the typical React setup. However, that's not the only data flow going on.  

We need to have the child have a reference callback function that will set a class variable of the parent, which I named `divElement`.   
You'll notice each child component has a `ref={props.divRef}`.  This ref variable points to the function represented by `props.divRef`: `DivReference`.
The function parameter in a ref callback is the reference to self.  I have named this parameter `el`, short for element.  Since the `ref=` is included in the child component's div, the `el` parameter refers to this div.  The function `DivReference` then sets the reference to this div equal to the parent's class variable, `divElement`.  Now, `divElement` knows exactly which of the children was rendered!

Note: divs are not usually focusable.  To make a focusable div, I needed to set its `tabIndex="0"`.  The zero value ensures it is focusable, and if a user were to press tab on the website, this div would become focusable only after other focusable elements that appear before it in the document source order were focused.

Okay great, the parent now has a handle on the specific child that will be rendered.  It needs to issue the command to focus it.  We do that in `componentDidUpdate`.  You'll see in that function we call `this.divElement.focus();` and boom it is automatically focused.  No more clicking. 

For the official react documentation on refs, be sure to check out the [literature](https://reactjs.org/docs/refs-and-the-dom.html).
