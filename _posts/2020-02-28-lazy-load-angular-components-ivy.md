---
layout: post
title: "Lazy-Load Angular Components with Ivy"
categories: angular
tags: [angular]
---

At the time of its creation, Angular apps load all modules immediately irrespective of their utility and usage in the app. Enter lazy loading. To reduce the number of components the app needs to load and save on resources such as processor power and network bandwidth, we implement the lazy loading pattern in the app. Lazy loading helps keep initial bundle sizes smaller, which in turn helps decrease load times.  
Lazy loading of single components has not been possible so far. However, with Angular Ivy (or just Ivy), more abilities are now possible.

## Locality in Ivy

Modules are the main building blocks of Angular. These contain several components, directives, pipes, and services. In layman’s terms, today’s application cannot exist without these modules.  

Ivy introduces the concept of ‘Locality’ in which a component can exist without a module. With Locality, all the metadata is local to that particular component.  
A real-world application of Lazy Loading of Component  
Create a new application using Angular CLI. Our app.component.ts file will look like this:

```ts
import { Component } from '@angular/core';
@Component({
	selector: 'app-component',
	templateUrl: './app.component.html',
	styleUrls: ['./app.component.scss']
})
export class AppComponent {
	title = 'my-app';
}
```
```ts
ng g greet  — flat — skip-import
```

In the above command, skip-import will not import the component in the current module. We will load this component inside the module using lazy load.

```ts
import { Component, OnInit } from '@angular/core';
@Component({
  selector: 'greet',
  templateUrl: './greet.component.html',
  styleUrls: ['./greet.component.scss']
})
export class GreetComponent implements OnInit {
  constructor() { }
  ngOnInit() {
  }
}
```
We see that the greet component is created and we will pass data to this component,

```ts
@Input() message: string;
@Output() sendEventEmmiter = new EventEmitter()
```
For now, this is sufficient for the greet component. We will now lazy load the greet component in the  _app.component_. To do this, we have to inject

viewContainerRef and ComponentFactoryResolver
constructor(private vcrf: ViewContainerRef, private cfr: ComponentFactoryResolver) { }

What is  _viewContainerRef_? – This is a container in which one or more views can be attached  
What is  _ComponentFactoryResolver_? – This is a simple registry that maps  _Components_  to generated  _ComponentFactory_  classes.  
These classes can be used to create instances of components. Use these to obtain the factory for a given component type, then use the factory’s  _create()_  method to create a component of that type.

## Create the method

Now create the method to lazy load the component in  _app.component_

```ts
loadComponent() {
  this.vcrf.clear();
  import('./greet/greet.component').then(({ GreetComponent }) => {
    let greetComp = this.vcrf.createComponent(
      this.cfr.resolveComponentFactory(GreetComponent)
    );
  });
}
```
Here we are using promises of JavaScript. These allow us to load the component only if the promise returns successfully. We are also using  _viewContaierRef_, and it’s method  _createComponent()_  in which we are passing  _componentFactoryResolver_. These should be able to create the component and host inside that particular  _viewContainerRef_.  
If you see, we still are not passing any value to the  _greetComponent_. In the above code, you will notice that we have a variable called  _greetComp_. This variable contains a reference to a lazy loaded component that we can use to pass data.

```ts
greetComp.instance.message = 'I am from app.component';
greetComp.instance.sendEventEmmiter.subscribe(data => {
  console.log(data);
});
```

When we combine all the above code so then our  _loadComponent()_  method will look like this,

```ts
loadComponent() {
  this.greetViewchildRef.clear();
  import('./greet/greet.component').then(({ GreetComponent }) => {
    let greetComp = this.greetViewchildRef.createComponent(
      this.cfr.resolveComponentFactory(GreetComponent)
    );
    greetComp.instance.message = 'I am from app.component';
    greetComp.instance.sendEventEmmiter.subscribe(data => {
      console.log(data);
    });
  });
}
```

## Displaying the result

To display the result, we have to load the  _greetComponent_  in the  _app.component.html_  file.

```html
<div>
  <ng-template #greetComp>
  </ng-template>
</div>
<button (click)="loadComponent()">Load component</button>
```

Here we use the  _ng-template_  to load the  _greetComponent_. We also need to add  _viewChild()_  to show the component data.

```ts
@ViewChild('greetComp', {read: ViewContainerRef})
private greetViewchildRef: ViewContainerRef
```
Note that if we want to use any module, then we need to add the decorator to the component itself, and then import the module within it.

```ts
@NgModule({
  declarations: [GreetComponent],
  imports: [FormsModule]
})
class GreetComponentModule {}
```
## Conclusion

We can create the component using a normal flow. However, what if we want to use the created component only for a specific action? If we use the usual way, we can reduce our application load by using a lazy load component.
