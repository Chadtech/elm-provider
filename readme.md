# Elm Provider

In Redux, the common pattern is for your UI modules to come paired as a component and a container. The component does the actual html rendering, while the container constructs `props` from the applications main `state`. You could imagine that looks like this..

```js
// ./greeter/container.js

import Greeter from './component.js'

const mapStateToProps = state => {
    name: [ 
        state.firstName, 
        state.lastName 
    ].join("")
};

export default connect(mapStateToProps, Greeter)

// ./greeter/component.js

export default const ({name}) =>
    (<p>{["Hello", name ].join(" ")}</p>)

```

What the UI component wants is a name to greet, but we need to construct that name from information in state, because the state doenst just have a thing in it called name.

Personally, I dont really think this component/container pattern is the way to go, but there is a real problem its trying to solve. When you have a big application, and your UI components are "very far" from your main model, and require information that different from what exists literally in the main model, it makes sense to explicitly define what the component needs in contrast to what the main model has and have an explicit transition step between your model and your component props. I have met a few people who tried Elm coming from Redux who said they miss the container/component stuff in Redux. This package is for them, to see if what they want is possible in Elm.

# Whats in this Package?

There is a module called `Html.Provider` and its identical to the standard `Html` module, except for a few features:
1. The `Html` type in `Html.Provider` is `Html model msg`, where `model` is the data type your components take.
2. Theres `Html.Provider.connect` which takes a `props -> Html model msg` function and a function that maps `model -> props`. 

Poke around in the example project in `./src` to see how it works. I tried to copy the Redux naming scheme directly. Heres maybe the clearest example of what this package provides:

Container
```elm
module Header.Container
    exposing
        ( container
        )

import Header.View exposing (Props)
import Html.Provider as Provider exposing (Html)
import Model exposing (Model)
import Msg exposing (Msg)


mapStateToProps : Model -> Props
mapStateToProps model =
    { greeting =
        if String.isEmpty model.name then
            "Hello you!"
        else
            [ "Hello "
            , model.name
            , "!"
            ]
                |> String.concat
    }


container : Html Model Msg
container =
    Provider.connect
        Header.View.view
        mapStateToProps

```
Component
```elm
module Header.View
    exposing
        ( Props
        , view
        )

import Html.Provider as Html exposing (Html, div, p)
import Model exposing (Model)
import Msg exposing (Msg)


type alias Props =
    { greeting : String }


view : Props -> Html Model Msg
view props =
    p
        []
        [ Html.text props.greeting ]
```