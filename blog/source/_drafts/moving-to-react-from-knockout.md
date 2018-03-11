

Part of the allure of AllReady has been the fact that we've got a good set of libraries and services that are current and valuable to gain experience in. Knockout has been a stalwart, but one of the original authors has moved on from the original project and as a library it is just not gaining the popularity that other libraries (like Angular, React and Vue) are garnishing.

Angular is a more complete application framework and wants to own things like routing, service communication and the like. It has dependecy injection wired in and you quickly have to assume the same opinions about project structure, module organization and service implementation if you want to succeed. For this reason, integrating it into an existing application with a lot of server-side rendering is not ideal.

That still leaves us with several options, and given the rise in popularity of React and the strong community behind it, we really wanted to make our project a compelling choice for anyone looking to ramp up their client-side skill set. And to be completely transparent, we also want to be able to draw from the biggest pool of developers possible.

To do the conversion we started by adding react to our project.

`npm install react --save`
`npm install @types/react  --save`

We also need the other react-related project which helps the library talk to the DOM.

`npm install react-dom  --save`
`npm install @types/react-dom --save`

We then added a script reference in the view, though we'll likely pull this out later to the shared view "_layout.cshtml" so that react is available everywhere.

## Upgrading TypeScript

The project we were working with actually had some TypeScript in it, though it was not extensively used. To get us on the latest version we ran `npm` again:

`npm install typescript@latest --save`

As a side note, knowing that we have to do all of those installs, we can shortcut our command to the following:



`npm install react @types/react react-dom @types/react-dom typescript@latest --save`

## So, This is a little different

One of the first things I read about React - and I'm paraphrasing a bit here - ws that React is different because, unlike other JavaScript libraries that put JS into your HTML, React puts HTML into your JavaScript. Basically, we want to make template and binding easier, and this is where JSX comes in. It's an embeddable sytax that muxes up your markup and script and lets you flow between JS and HTML. 

To leverage this feature we needed to tweak our settings in our `tsconfig.json`, setting the `jsx` property to `react`. You can read more about the compiler options [here](https://www.typescriptlang.org/docs/handbook/jsx.html). Our TypeScript files that in which we wish to use JSX will end with the `.jsx` file extension.  

We also updated the `include` object in `tsconfig.json` to include our `*.tsx` files, and we changed our gulpfile to include the different JavaScript and TypeScript files that we'd like output into our `wwwroot`.

The view we started with was our `Index` method on our `Manage` controller. This view has a minimal amount of data binding syntax and presents a small enough feature that we could tackle in our first sitting. 

The feature we wanted to replace allowed a user to add skills to their profile. Skills help administrators match volunteers to outstanding tasks.  We added a `SkillPicker.tsx` file to the solution and built out the template.

```
// template
```

To see the changes in our view, we add a little bit of code so that our script is executed by the browser.  The first thing to do is to give a place for our react components to render. We put this in our document:

```
// view code
```

Then, we included the required scripts in our view:
```
/// scripts
```

Finally, we use the command line to build the TSX and render the final JS that will be added to our view.

`gulp build`

## Rendering the Data
Obviously we wanted more than just a 'Hello World' style application. When we look at how the existing feature was laid out, there were a couple of components that stood out. 



We chose to break it down into the following pieces:

1. SkillPicker
1. DeleteSkill
1. 

The data that backs the view is loaded as JSON to the view, and there are a few ways properties of that data needs to surface. We added a few classes to represent that data:

1. CurrentSkill
1. AvailableSkill

## Reimplementing the Original Features

The UI allowed users to drop skills, and so we needed to preserve this functionality as well. We built out the class a little further to include a delete method that removed the chosen skill from the collection.

```
// delete method
```

## Peripheral Benefits

The beauty of these changes was that we also simplfied our view, moving components to separate files and seeing more clearly how the data binding worked. React seems to feel like it's more closely part of the DOM. 

## Next Steps


