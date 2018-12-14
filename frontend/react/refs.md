# Everything you need to know about using Refs

(Note: discussions below assumes React v16.3 or higher is used)

### Why do we need it?
React components wrap up DOM element to give encapsulations:

```js
function FancyButton(props) {
  return (
    <button className="FancyButton">
      {props.children}
    </button>
  );
}

// We render it like this and users don't know what is actually being rendered without using tools like Chrome 
// DevTools to inspect

// <FancyButton>Submit</FancyButton>
```

But sometimes, we need to be able to access underlying DOM i.e manage button focus, attach accessbility attrs.
Thus, we need a way to reference DOM elements.


### How can we access it?

DOM is not always accessible via using `Ref`. Below, we will discuss a few different scenarios where DOM can be accessed. Besides, the case where DOM isn't accessible.

  * Access DOM from inside a class component
    * Ref assigned to a DOM
  
      ```js
        // React will assign the current property with the DOM element when the component mounts, and assign it back to null when it unmounts. ref 
        // updates happen before componentDidMount or componentDidUpdate lifecycle methods.
        class MyComponent extends React.Component {
          constructor(props) {
            super(props);
            this.textInput = React.createRef();
          }
          
          focusTextInput = () => {
            console.log(this.textInput.current); // <input type="text">
            // input is available by calling this.textInput.current
            this.textInput.current.focus();
          }
          
          render() {
            return (
              <div>
                <input
                  type="text"
                  ref={this.textInput} />

                <input
                  type="button"
                  value="Focus the text input"
                  onClick={this.focusTextInput}
                />
              </div>
            )
          }
        }
      ```
  
    * Ref assigned to a class component (where innerRef comes into play)
      ```js
        class CustomTextInput extends React.Component {
          render() {
            return <input type="text" ref={this.props.innerRef} />
          }
        }

        class MyComponent extends React.Component {
          constructor(props) {
            super(props);
            this.textInput = React.createRef();
          }

          focusTextInput = () => {
            this.textInput.current.focus();
          }

          render() {
            return (
              <div>
                <CustomTextInput innerRef={this.textInput} />
                <input
                  type="button"
                  value="Focus the text input"
                  onClick={this.focusTextInput}
                />
              </div>
            );
          }
        } 
      ```
      
     * Ref will not work against function component
      
      ```js
        function MyFunctionComponent() {
          return <input />;
        }

        class Parent extends React.Component {
          constructor(props) {
            super(props);
            this.textInput = React.createRef();
          }
          render() {
            // This will *not* work!
            return (
              <MyFunctionComponent ref={this.textInput} />
            );
          }
        }      
      ```

  * Access DOM from inside a function component
    Both snippets below work:
    
    * Ref assigned to a DOM
    
      ```js
        function CustomTextInput(props) {
          // textInput must be declared here so the ref can refer to it
          let textInput = React.createRef();

          function handleClick() {
            textInput.current.focus();
          }

          return (
            <div>
              <input
                type="text"
                ref={textInput} />

              <input
                type="button"
                value="Focus the text input"
                onClick={handleClick}
              />
            </div>
          );
        }    
      ```
      
    * Ref assigned to a class component
    
      ```js
        class CustomTextInput extends React.Component {
          render() {
            return <input type="text" ref={this.props.innerRef} />
          }
        }      
      
        function MyComponent(props) {
          // textInput must be declared here so the ref can refer to it
          let textInput = React.createRef();

          function handleClick() {
            textInput.current.focus();
          }

          return (
            <div>
              <CustomTextInput innerRef={textInput} />
              <input
                type="button"
                value="Focus the text input"
                onClick={handleClick}
              />
            </div>
          );
        }    
      ```
      
    
    
    
    
  
   