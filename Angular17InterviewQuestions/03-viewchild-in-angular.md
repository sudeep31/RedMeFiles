# 🎯 ViewChild in Angular: Complete Guide

## 🎯 **Question Overview**

_"What is ViewChild in Angular?"_

## 🔍 **Understanding ViewChild**

`ViewChild` is a **decorator** in Angular that allows you to access and manipulate DOM elements, child components, or directives from within a parent component class. It's one of the most powerful features for component interaction and DOM manipulation.

Think of ViewChild as your **direct line of communication** with elements in your component's template!

## 🛠️ **Basic Syntax & Usage**

### **Basic ViewChild Declaration**

```typescript
import { Component, ViewChild, ElementRef, AfterViewInit } from "@angular/core";

@Component({
  selector: "app-parent",
  template: `
    <input #inputRef type="text" placeholder="Enter your name" />
    <button (click)="focusInput()">Focus Input</button>
  `,
})
export class ParentComponent implements AfterViewInit {
  @ViewChild("inputRef") inputElement!: ElementRef;

  ngAfterViewInit() {
    console.log("Input element:", this.inputElement.nativeElement);
  }

  focusInput() {
    this.inputElement.nativeElement.focus();
  }
}
```

### **ViewChild with Component Reference**

```typescript
// Child Component
@Component({
  selector: "app-child",
  template: `
    <div>
      <h3>Child Component</h3>
      <p>Counter: {{ counter }}</p>
    </div>
  `,
})
export class ChildComponent {
  counter = 0;

  increment() {
    this.counter++;
  }

  reset() {
    this.counter = 0;
  }
}

// Parent Component
@Component({
  selector: "app-parent",
  template: `
    <app-child></app-child>
    <button (click)="incrementChild()">Increment Child</button>
    <button (click)="resetChild()">Reset Child</button>
  `,
})
export class ParentComponent implements AfterViewInit {
  @ViewChild(ChildComponent) childComponent!: ChildComponent;

  ngAfterViewInit() {
    console.log("Child component loaded:", this.childComponent);
  }

  incrementChild() {
    this.childComponent.increment();
  }

  resetChild() {
    this.childComponent.reset();
  }
}
```

## 🔧 **Advanced ViewChild Usage**

### **1. ViewChild with Directive**

```typescript
// Custom Directive
@Directive({
  selector: "[appHighlight]",
})
export class HighlightDirective {
  constructor(private el: ElementRef) {}

  highlight(color: string) {
    this.el.nativeElement.style.backgroundColor = color;
  }

  removeHighlight() {
    this.el.nativeElement.style.backgroundColor = "";
  }
}

// Component using the directive
@Component({
  selector: "app-example",
  template: `
    <div appHighlight>This div can be highlighted</div>
    <button (click)="highlightDiv()">Highlight</button>
    <button (click)="removeHighlight()">Remove Highlight</button>
  `,
})
export class ExampleComponent implements AfterViewInit {
  @ViewChild(HighlightDirective) highlightDirective!: HighlightDirective;

  ngAfterViewInit() {
    console.log("Highlight directive ready:", this.highlightDirective);
  }

  highlightDiv() {
    this.highlightDirective.highlight("yellow");
  }

  removeHighlight() {
    this.highlightDirective.removeHighlight();
  }
}
```

### **2. ViewChild with Static Option**

```typescript
@Component({
  selector: "app-static-example",
  template: `
    <div #staticElement>Always visible element</div>
    <div #dynamicElement *ngIf="showDynamic">Dynamic element</div>
    <button (click)="toggleDynamic()">Toggle Dynamic</button>
  `,
})
export class StaticExampleComponent implements OnInit, AfterViewInit {
  // Static: true - available in ngOnInit
  @ViewChild("staticElement", { static: true })
  staticElement!: ElementRef;

  // Static: false (default) - available in ngAfterViewInit
  @ViewChild("dynamicElement", { static: false })
  dynamicElement?: ElementRef;

  showDynamic = false;

  ngOnInit() {
    // Static element is available here
    console.log("Static element in ngOnInit:", this.staticElement);

    // Dynamic element is NOT available here (undefined)
    console.log("Dynamic element in ngOnInit:", this.dynamicElement);
  }

  ngAfterViewInit() {
    // Both elements are available here (if dynamic element exists)
    console.log("Static element in ngAfterViewInit:", this.staticElement);
    console.log("Dynamic element in ngAfterViewInit:", this.dynamicElement);
  }

  toggleDynamic() {
    this.showDynamic = !this.showDynamic;
  }
}
```

### **3. ViewChild with Read Option**

```typescript
@Component({
  selector: "app-read-example",
  template: ` <input #inputRef type="text" value="Hello World" /> `,
})
export class ReadExampleComponent implements AfterViewInit {
  // Read as ElementRef (default)
  @ViewChild("inputRef") inputAsElementRef!: ElementRef;

  // Read as HTMLInputElement
  @ViewChild("inputRef", { read: ElementRef })
  inputAsElement!: ElementRef<HTMLInputElement>;

  // Read ViewContainerRef (useful for dynamic content)
  @ViewChild("inputRef", { read: ViewContainerRef })
  inputAsViewContainer!: ViewContainerRef;

  ngAfterViewInit() {
    console.log("ElementRef:", this.inputAsElementRef.nativeElement.value);
    console.log("HTML Element:", this.inputAsElement.nativeElement.value);
    console.log("ViewContainerRef:", this.inputAsViewContainer);
  }
}
```

## 🚀 **Real-World Examples**

### **1. Modal Component Control**

```typescript
// Modal Component
@Component({
  selector: "app-modal",
  template: `
    <div class="modal" [class.show]="isVisible">
      <div class="modal-content">
        <span class="close" (click)="close()">&times;</span>
        <ng-content></ng-content>
      </div>
    </div>
  `,
  styles: [
    `
      .modal {
        display: none;
      }
      .modal.show {
        display: block;
      }
      .modal-content {
        background: white;
        padding: 20px;
      }
    `,
  ],
})
export class ModalComponent {
  isVisible = false;

  open() {
    this.isVisible = true;
    document.body.style.overflow = "hidden";
  }

  close() {
    this.isVisible = false;
    document.body.style.overflow = "auto";
  }
}

// Parent Component
@Component({
  selector: "app-parent",
  template: `
    <button (click)="openModal()">Open Modal</button>

    <app-modal>
      <h2>Modal Content</h2>
      <p>This is some content in the modal.</p>
      <button (click)="closeModal()">Close Modal</button>
    </app-modal>
  `,
})
export class ParentComponent {
  @ViewChild(ModalComponent) modal!: ModalComponent;

  openModal() {
    this.modal.open();
  }

  closeModal() {
    this.modal.close();
  }
}
```

### **2. Form Validation Control**

```typescript
@Component({
  selector: "app-form-example",
  template: `
    <form #userForm="ngForm" (ngSubmit)="onSubmit()">
      <input
        #nameInput="ngModel"
        name="name"
        [(ngModel)]="user.name"
        required
        minlength="2"
        placeholder="Name"
      />
      <div *ngIf="nameInput.invalid && nameInput.touched" class="error">
        Name is required and must be at least 2 characters
      </div>

      <input
        #emailInput="ngModel"
        name="email"
        [(ngModel)]="user.email"
        required
        email
        placeholder="Email"
      />
      <div *ngIf="emailInput.invalid && emailInput.touched" class="error">
        Please enter a valid email
      </div>

      <button type="submit" [disabled]="userForm.invalid">Submit</button>
      <button type="button" (click)="resetForm()">Reset</button>
    </form>
  `,
})
export class FormExampleComponent {
  @ViewChild("userForm") form!: NgForm;
  @ViewChild("nameInput") nameInput!: NgModel;
  @ViewChild("emailInput") emailInput!: NgModel;

  user = { name: "", email: "" };

  onSubmit() {
    if (this.form.valid) {
      console.log("Form submitted:", this.user);
    }
  }

  resetForm() {
    this.form.reset();
    this.user = { name: "", email: "" };
  }

  validateName() {
    return this.nameInput.valid;
  }
}
```

### **3. Chart Component Integration**

```typescript
@Component({
  selector: "app-chart",
  template: `
    <div>
      <canvas #chartCanvas width="400" height="200"></canvas>
      <button (click)="updateChart()">Update Chart</button>
      <button (click)="downloadChart()">Download Chart</button>
    </div>
  `,
})
export class ChartComponent implements AfterViewInit {
  @ViewChild("chartCanvas") chartCanvas!: ElementRef<HTMLCanvasElement>;

  private chart: any; // Assume Chart.js or similar

  ngAfterViewInit() {
    this.initializeChart();
  }

  initializeChart() {
    const ctx = this.chartCanvas.nativeElement.getContext("2d");
    // Initialize your chart here
    console.log("Chart initialized on canvas:", ctx);
  }

  updateChart() {
    // Update chart data
    console.log("Updating chart...");
  }

  downloadChart() {
    const canvas = this.chartCanvas.nativeElement;
    const dataURL = canvas.toDataURL("image/png");

    const link = document.createElement("a");
    link.download = "chart.png";
    link.href = dataURL;
    link.click();
  }
}
```

## 🔄 **ViewChild vs ViewChildren**

```typescript
@Component({
  selector: "app-multiple-elements",
  template: `
    <div *ngFor="let item of items; let i = index">
      <input #itemInput [value]="item" (input)="updateItem(i, $event)" />
    </div>
    <button (click)="focusFirstInput()">Focus First</button>
    <button (click)="focusAllInputs()">Focus All</button>
    <button (click)="clearAllInputs()">Clear All</button>
  `,
})
export class MultipleElementsComponent implements AfterViewInit {
  @ViewChild("itemInput") firstInput!: ElementRef; // Gets only the first one
  @ViewChildren("itemInput") allInputs!: QueryList<ElementRef>; // Gets all

  items = ["Item 1", "Item 2", "Item 3"];

  ngAfterViewInit() {
    console.log("First input:", this.firstInput);
    console.log("All inputs:", this.allInputs.toArray());

    // Listen for changes in the QueryList
    this.allInputs.changes.subscribe(() => {
      console.log("Input list changed:", this.allInputs.toArray());
    });
  }

  focusFirstInput() {
    this.firstInput.nativeElement.focus();
  }

  focusAllInputs() {
    this.allInputs.forEach((input) => {
      input.nativeElement.focus();
    });
  }

  clearAllInputs() {
    this.allInputs.forEach((input, index) => {
      input.nativeElement.value = "";
      this.items[index] = "";
    });
  }

  updateItem(index: number, event: any) {
    this.items[index] = event.target.value;
  }
}
```

## 🚨 **Common Pitfalls & Best Practices**

### **❌ Common Mistakes**

1. **Accessing ViewChild in ngOnInit**

```typescript
// ❌ Wrong - ViewChild not available yet
export class BadComponent implements OnInit {
  @ViewChild("element") element!: ElementRef;

  ngOnInit() {
    // This will be undefined!
    console.log(this.element); // undefined
  }
}
```

2. **Not handling undefined ViewChild**

```typescript
// ❌ Wrong - Can cause runtime errors
focusElement() {
  this.element.nativeElement.focus(); // Error if element doesn't exist
}
```

### **✅ Best Practices**

```typescript
// ✅ Correct - Access ViewChild in ngAfterViewInit
export class GoodComponent implements AfterViewInit {
  @ViewChild("element") element?: ElementRef;

  ngAfterViewInit() {
    // ViewChild is available here
    console.log(this.element); // ElementRef
  }

  focusElement() {
    // Always check if element exists
    if (this.element) {
      this.element.nativeElement.focus();
    }
  }
}
```

## 🔧 **Testing ViewChild**

```typescript
import { ComponentFixture, TestBed } from "@angular/core/testing";
import { By } from "@angular/platform-browser";

describe("ParentComponent", () => {
  let component: ParentComponent;
  let fixture: ComponentFixture<ParentComponent>;

  beforeEach(() => {
    TestBed.configureTestingModule({
      declarations: [ParentComponent, ChildComponent],
    });

    fixture = TestBed.createComponent(ParentComponent);
    component = fixture.componentInstance;
  });

  it("should access child component via ViewChild", () => {
    fixture.detectChanges(); // Trigger ngAfterViewInit

    expect(component.childComponent).toBeDefined();
    expect(component.childComponent.counter).toBe(0);
  });

  it("should call child component methods", () => {
    fixture.detectChanges();

    spyOn(component.childComponent, "increment");

    component.incrementChild();

    expect(component.childComponent.increment).toHaveBeenCalled();
  });
});
```

## 📊 **Angular Version Comparison**

| Feature                   | Angular 15            | Angular 17-19               |
| ------------------------- | --------------------- | --------------------------- |
| **ViewChild Decorator**   | Same syntax           | Same syntax                 |
| **Static Option**         | Available             | Available                   |
| **Read Option**           | Available             | Available                   |
| **Type Safety**           | Good with strict mode | Enhanced TypeScript support |
| **Signal Integration**    | Not available         | Can work with signals       |
| **Standalone Components** | Limited support       | Full support                |

### **Modern Angular 19 with Signals**

```typescript
// Angular 19 with Signals
import {
  Component,
  ViewChild,
  ElementRef,
  AfterViewInit,
  signal,
} from "@angular/core";

@Component({
  selector: "app-modern",
  template: `
    <input #inputRef type="text" [value]="inputValue()" />
    <button (click)="updateValue()">Update</button>
    <p>Current value: {{ inputValue() }}</p>
  `,
  standalone: true,
})
export class ModernComponent implements AfterViewInit {
  @ViewChild("inputRef") inputElement!: ElementRef;

  inputValue = signal("Initial value");

  ngAfterViewInit() {
    // Can integrate ViewChild with signals
    this.inputElement.nativeElement.addEventListener("input", (event: any) => {
      this.inputValue.set(event.target.value);
    });
  }

  updateValue() {
    const newValue = "Updated at " + new Date().toLocaleTimeString();
    this.inputValue.set(newValue);
    this.inputElement.nativeElement.value = newValue;
  }
}
```

## 🎯 **Key Takeaways**

1. **ViewChild provides direct access** to DOM elements, child components, and directives
2. **Use ngAfterViewInit** for ViewChild access, not ngOnInit
3. **Always check for undefined** when accessing ViewChild properties
4. **Use static: true** only for elements that are always present
5. **ViewChildren for multiple elements** with QueryList
6. **Great for component communication** and DOM manipulation
7. **Essential for integrating third-party libraries** that need direct DOM access

ViewChild is a powerful tool that bridges the gap between Angular's component architecture and direct DOM manipulation when needed! 🚀
