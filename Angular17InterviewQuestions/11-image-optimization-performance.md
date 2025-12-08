# 🖼️ Optimizing Large Image Loads for Performance in Angular

## 🎯 **Question Overview**

_"How do you optimize large image loads from performance aspect?"_

## 🔍 **Understanding Image Performance Challenges**

Large images can significantly impact your Angular application's performance by:

- **Slowing down initial page loads** 🐌
- **Consuming excessive bandwidth** 📡
- **Causing layout shifts** during loading 📐
- **Degrading user experience** on slow connections 🌐

Let's explore comprehensive strategies to optimize image loading!

## 🚀 **Modern Angular Image Optimization Strategies**

### **1. 🖼️ NgOptimizedImage Directive (Angular 15+)**

Angular's built-in image optimization directive:

```typescript
// Import NgOptimizedImage
import { NgOptimizedImage } from "@angular/common";

@Component({
  selector: "app-image-gallery",
  template: `
    <div class="gallery">
      <!-- Basic optimized image -->
      <img
        ngSrc="assets/images/hero-banner.jpg"
        alt="Hero Banner"
        width="1200"
        height="600"
        priority
      />

      <!-- Responsive image with breakpoints -->
      <img
        ngSrc="assets/images/product-{{ product.id }}.jpg"
        [alt]="product.name"
        width="400"
        height="300"
        sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
      />

      <!-- Lazy loaded image -->
      <img
        ngSrc="assets/images/gallery-{{ item.id }}.jpg"
        [alt]="item.description"
        width="300"
        height="200"
        loading="lazy"
      />
    </div>
  `,
  standalone: true,
  imports: [NgOptimizedImage, CommonModule],
})
export class ImageGalleryComponent {
  product = { id: "123", name: "Amazing Product" };
  galleryItems = [
    { id: "1", description: "Beautiful landscape" },
    { id: "2", description: "City skyline" },
  ];
}
```

### **2. 🔄 Progressive Image Loading Service**

```typescript
// progressive-image.service.ts
@Injectable({
  providedIn: "root",
})
export class ProgressiveImageService {
  private imageCache = new Map<string, string>();
  private loadingImages = new Map<string, Observable<string>>();

  loadImageProgressively(
    src: string,
    placeholderSrc?: string
  ): Observable<ImageLoadState> {
    return new Observable((observer) => {
      // Emit placeholder immediately
      if (placeholderSrc) {
        observer.next({
          src: placeholderSrc,
          state: "placeholder",
          progress: 0,
        });
      }

      // Check cache first
      if (this.imageCache.has(src)) {
        observer.next({
          src: this.imageCache.get(src)!,
          state: "loaded",
          progress: 100,
        });
        observer.complete();
        return;
      }

      // Check if already loading
      const existingLoad = this.loadingImages.get(src);
      if (existingLoad) {
        existingLoad.subscribe({
          next: (loadedSrc) =>
            observer.next({
              src: loadedSrc,
              state: "loaded",
              progress: 100,
            }),
          error: (err) => observer.error(err),
          complete: () => observer.complete(),
        });
        return;
      }

      // Start loading
      const loadObservable = this.loadImageWithProgress(src);
      this.loadingImages.set(src, loadObservable);

      loadObservable.subscribe({
        next: (result) => {
          if (result.progress < 100) {
            observer.next({
              src: placeholderSrc || src,
              state: "loading",
              progress: result.progress,
            });
          } else {
            this.imageCache.set(src, result.src);
            this.loadingImages.delete(src);
            observer.next({
              src: result.src,
              state: "loaded",
              progress: 100,
            });
            observer.complete();
          }
        },
        error: (error) => {
          this.loadingImages.delete(src);
          observer.error(error);
        },
      });
    });
  }

  private loadImageWithProgress(
    src: string
  ): Observable<{ src: string; progress: number }> {
    return new Observable((observer) => {
      const xhr = new XMLHttpRequest();

      xhr.open("GET", src);
      xhr.responseType = "blob";

      xhr.onprogress = (event) => {
        if (event.lengthComputable) {
          const progress = (event.loaded / event.total) * 100;
          observer.next({ src, progress });
        }
      };

      xhr.onload = () => {
        if (xhr.status === 200) {
          const blob = xhr.response;
          const objectUrl = URL.createObjectURL(blob);
          observer.next({ src: objectUrl, progress: 100 });
          observer.complete();
        } else {
          observer.error(new Error(`Failed to load image: ${xhr.status}`));
        }
      };

      xhr.onerror = () => {
        observer.error(new Error("Network error while loading image"));
      };

      xhr.send();
    });
  }

  preloadImages(urls: string[]): Observable<string[]> {
    const preloadObservables = urls.map((url) =>
      this.loadImageWithProgress(url).pipe(
        map((result) => result.src),
        take(1),
        catchError(() => of(url)) // Return original URL on error
      )
    );

    return forkJoin(preloadObservables);
  }

  clearCache(): void {
    // Clean up object URLs to prevent memory leaks
    this.imageCache.forEach((url) => {
      if (url.startsWith("blob:")) {
        URL.revokeObjectURL(url);
      }
    });
    this.imageCache.clear();
  }
}

interface ImageLoadState {
  src: string;
  state: "placeholder" | "loading" | "loaded" | "error";
  progress: number;
}
```

### **3. 🎨 Progressive Image Component**

```typescript
// progressive-image.component.ts
@Component({
  selector: "app-progressive-image",
  template: `
    <div
      class="progressive-image"
      [class.loading]="imageState.state === 'loading'"
      [class.loaded]="imageState.state === 'loaded'"
      [style.aspect-ratio]="aspectRatio"
    >
      <!-- Base64 placeholder or low-res image -->
      <img
        *ngIf="placeholder"
        [src]="placeholder"
        [alt]="alt"
        class="placeholder"
        [style.filter]="
          imageState.state === 'loaded' ? 'blur(0px)' : 'blur(10px)'
        "
      />

      <!-- Main image -->
      <img
        [src]="imageState.src"
        [alt]="alt"
        class="main-image"
        [class.visible]="imageState.state === 'loaded'"
        (load)="onImageLoad()"
        (error)="onImageError()"
      />

      <!-- Loading indicator -->
      <div *ngIf="imageState.state === 'loading'" class="loading-overlay">
        <div class="progress-bar">
          <div
            class="progress-fill"
            [style.width.%]="imageState.progress"
          ></div>
        </div>
        <span class="loading-text"
          >{{ imageState.progress | number : "1.0-0" }}%</span
        >
      </div>

      <!-- Error state -->
      <div *ngIf="imageState.state === 'error'" class="error-overlay">
        <span class="error-icon">❌</span>
        <span class="error-text">Failed to load image</span>
        <button (click)="retryLoad()" class="retry-button">Retry</button>
      </div>
    </div>
  `,
  styles: [
    `
      .progressive-image {
        position: relative;
        overflow: hidden;
        background-color: #f0f0f0;
        display: flex;
        align-items: center;
        justify-content: center;
      }

      .placeholder {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        object-fit: cover;
        transition: filter 0.3s ease;
      }

      .main-image {
        width: 100%;
        height: 100%;
        object-fit: cover;
        opacity: 0;
        transition: opacity 0.3s ease;
      }

      .main-image.visible {
        opacity: 1;
      }

      .loading-overlay {
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        background: rgba(255, 255, 255, 0.9);
        padding: 20px;
        border-radius: 8px;
        text-align: center;
        backdrop-filter: blur(4px);
      }

      .progress-bar {
        width: 200px;
        height: 4px;
        background: #e0e0e0;
        border-radius: 2px;
        overflow: hidden;
        margin-bottom: 10px;
      }

      .progress-fill {
        height: 100%;
        background: linear-gradient(90deg, #007acc, #00a8ff);
        transition: width 0.3s ease;
      }

      .error-overlay {
        position: absolute;
        inset: 0;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        background: rgba(255, 255, 255, 0.95);
        color: #d32f2f;
      }

      .error-icon {
        font-size: 2em;
        margin-bottom: 10px;
      }

      .retry-button {
        margin-top: 10px;
        padding: 8px 16px;
        background: #007acc;
        color: white;
        border: none;
        border-radius: 4px;
        cursor: pointer;
      }

      .retry-button:hover {
        background: #005999;
      }
    `,
  ],
  standalone: true,
  imports: [CommonModule],
})
export class ProgressiveImageComponent implements OnInit, OnDestroy {
  @Input() src!: string;
  @Input() placeholder?: string;
  @Input() alt = "";
  @Input() aspectRatio = "auto";
  @Input() enableProgressTracking = true;

  @Output() imageLoaded = new EventEmitter<void>();
  @Output() imageError = new EventEmitter<Error>();

  imageState: ImageLoadState = {
    src: "",
    state: "placeholder",
    progress: 0,
  };

  private destroy$ = new Subject<void>();

  constructor(private progressiveImageService: ProgressiveImageService) {}

  ngOnInit() {
    this.loadImage();
  }

  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }

  private loadImage() {
    if (this.enableProgressTracking) {
      this.progressiveImageService
        .loadImageProgressively(this.src, this.placeholder)
        .pipe(takeUntil(this.destroy$))
        .subscribe({
          next: (state) => (this.imageState = state),
          error: (error) => {
            this.imageState = {
              src: this.placeholder || "",
              state: "error",
              progress: 0,
            };
            this.imageError.emit(error);
          },
        });
    } else {
      // Simple loading without progress tracking
      this.loadSimple();
    }
  }

  private loadSimple() {
    this.imageState = {
      src: this.placeholder || "",
      state: "loading",
      progress: 0,
    };

    const img = new Image();
    img.onload = () => {
      this.imageState = {
        src: this.src,
        state: "loaded",
        progress: 100,
      };
      this.imageLoaded.emit();
    };
    img.onerror = () => {
      this.imageState = {
        src: this.placeholder || "",
        state: "error",
        progress: 0,
      };
      this.imageError.emit(new Error("Failed to load image"));
    };
    img.src = this.src;
  }

  onImageLoad() {
    if (this.imageState.state === "loaded") {
      this.imageLoaded.emit();
    }
  }

  onImageError() {
    if (this.imageState.state !== "error") {
      this.imageState = {
        src: this.placeholder || "",
        state: "error",
        progress: 0,
      };
      this.imageError.emit(new Error("Image load error"));
    }
  }

  retryLoad() {
    this.loadImage();
  }
}
```

### **4. 📱 Responsive Image Component**

```typescript
// responsive-image.component.ts
@Component({
  selector: "app-responsive-image",
  template: `
    <picture class="responsive-picture">
      <!-- WebP sources -->
      <source
        *ngFor="let breakpoint of webpBreakpoints"
        [media]="breakpoint.media"
        [srcset]="breakpoint.srcset"
        type="image/webp"
      />

      <!-- AVIF sources (modern browsers) -->
      <source
        *ngFor="let breakpoint of avifBreakpoints"
        [media]="breakpoint.media"
        [srcset]="breakpoint.srcset"
        type="image/avif"
      />

      <!-- Fallback image -->
      <img
        [src]="fallbackSrc"
        [alt]="alt"
        [loading]="lazy ? 'lazy' : 'eager'"
        [class]="imageClass"
        (load)="onLoad()"
        (error)="onError()"
      />
    </picture>
  `,
  styles: [
    `
      .responsive-picture {
        display: block;
        width: 100%;
        height: auto;
      }

      .responsive-picture img {
        width: 100%;
        height: auto;
        object-fit: cover;
      }
    `,
  ],
  standalone: true,
  imports: [CommonModule],
})
export class ResponsiveImageComponent implements OnInit {
  @Input() src!: string;
  @Input() alt = "";
  @Input() lazy = true;
  @Input() imageClass = "";
  @Input() breakpoints: ResponsiveBreakpoint[] = [];

  @Output() imageLoaded = new EventEmitter<void>();
  @Output() imageError = new EventEmitter<void>();

  webpBreakpoints: ResponsiveBreakpoint[] = [];
  avifBreakpoints: ResponsiveBreakpoint[] = [];
  fallbackSrc = "";

  ngOnInit() {
    this.setupResponsiveImages();
  }

  private setupResponsiveImages() {
    // Default breakpoints if none provided
    if (this.breakpoints.length === 0) {
      this.breakpoints = [
        { media: "(max-width: 480px)", size: "small" },
        { media: "(max-width: 768px)", size: "medium" },
        { media: "(max-width: 1200px)", size: "large" },
        { media: "(min-width: 1201px)", size: "xlarge" },
      ];
    }

    // Generate srcsets for different formats
    this.webpBreakpoints = this.breakpoints.map((bp) => ({
      ...bp,
      srcset: this.generateSrcset(bp.size, "webp"),
    }));

    this.avifBreakpoints = this.breakpoints.map((bp) => ({
      ...bp,
      srcset: this.generateSrcset(bp.size, "avif"),
    }));

    this.fallbackSrc = this.generateImageUrl("medium", "jpg");
  }

  private generateSrcset(size: string, format: string): string {
    const baseUrl = this.getBaseUrl();
    return [
      `${baseUrl}_${size}.${format} 1x`,
      `${baseUrl}_${size}@2x.${format} 2x`,
    ].join(", ");
  }

  private generateImageUrl(size: string, format: string): string {
    const baseUrl = this.getBaseUrl();
    return `${baseUrl}_${size}.${format}`;
  }

  private getBaseUrl(): string {
    // Remove file extension and size suffixes
    return this.src
      .replace(/\.(jpg|jpeg|png|webp|avif)$/i, "")
      .replace(/_[a-z]+(@\d+x)?$/i, "");
  }

  onLoad() {
    this.imageLoaded.emit();
  }

  onError() {
    this.imageError.emit();
  }
}

interface ResponsiveBreakpoint {
  media: string;
  size: string;
  srcset?: string;
}
```

### **5. 🔍 Intersection Observer Lazy Loading**

```typescript
// lazy-image.directive.ts
@Directive({
  selector: "img[appLazyLoad]",
  standalone: true,
})
export class LazyImageDirective implements OnInit, OnDestroy {
  @Input("appLazyLoad") src!: string;
  @Input() placeholder =
    "data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMSIgaGVpZ2h0PSIxIiB2aWV3Qm94PSIwIDAgMSAxIiBmaWxsPSJub25lIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPjxyZWN0IHdpZHRoPSIxIiBoZWlnaHQ9IjEiIGZpbGw9IiNGMEYwRjAiLz48L3N2Zz4=";
  @Input() rootMargin = "50px";
  @Input() threshold = 0.1;

  private observer?: IntersectionObserver;

  constructor(private el: ElementRef<HTMLImageElement>) {}

  ngOnInit() {
    // Set placeholder initially
    this.el.nativeElement.src = this.placeholder;
    this.el.nativeElement.classList.add("lazy-loading");

    // Setup intersection observer
    this.observer = new IntersectionObserver(
      (entries) => this.onIntersection(entries),
      {
        rootMargin: this.rootMargin,
        threshold: this.threshold,
      }
    );

    this.observer.observe(this.el.nativeElement);
  }

  ngOnDestroy() {
    if (this.observer) {
      this.observer.disconnect();
    }
  }

  private onIntersection(entries: IntersectionObserverEntry[]) {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        this.loadImage();
        this.observer?.unobserve(entry.target);
      }
    });
  }

  private loadImage() {
    const img = this.el.nativeElement;

    // Create a new image to preload
    const imageLoader = new Image();

    imageLoader.onload = () => {
      // Image loaded successfully
      img.src = this.src;
      img.classList.remove("lazy-loading");
      img.classList.add("lazy-loaded");
    };

    imageLoader.onerror = () => {
      // Handle error
      img.classList.remove("lazy-loading");
      img.classList.add("lazy-error");
    };

    imageLoader.src = this.src;
  }
}

// Usage in template
// <img appLazyLoad="path/to/large-image.jpg" alt="Description" placeholder="path/to/placeholder.jpg">
```

### **6. 🎯 Virtual Scrolling with Images**

```typescript
// virtual-image-list.component.ts
@Component({
  selector: "app-virtual-image-list",
  template: `
    <cdk-virtual-scroll-viewport
      itemSize="300"
      class="viewport"
      [style.height.px]="viewportHeight"
    >
      <div
        *cdkVirtualFor="let item of images; trackBy: trackByFn"
        class="image-item"
      >
        <app-progressive-image
          [src]="item.src"
          [placeholder]="item.placeholder"
          [alt]="item.alt"
          aspectRatio="16/9"
          (imageLoaded)="onImageLoaded(item.id)"
          (imageError)="onImageError(item.id)"
        >
        </app-progressive-image>

        <div class="image-info">
          <h3>{{ item.title }}</h3>
          <p>{{ item.description }}</p>
        </div>
      </div>
    </cdk-virtual-scroll-viewport>
  `,
  styles: [
    `
      .viewport {
        width: 100%;
        border: 1px solid #ccc;
      }

      .image-item {
        height: 300px;
        padding: 10px;
        border-bottom: 1px solid #eee;
        display: flex;
        gap: 15px;
      }

      .image-item app-progressive-image {
        flex: 0 0 400px;
      }

      .image-info {
        flex: 1;
        padding: 10px;
      }

      .image-info h3 {
        margin: 0 0 10px 0;
      }
    `,
  ],
  standalone: true,
  imports: [CommonModule, ScrollingModule, ProgressiveImageComponent],
})
export class VirtualImageListComponent implements OnInit {
  @Input() images: ImageItem[] = [];
  @Input() viewportHeight = 600;

  private loadedImages = new Set<string>();

  ngOnInit() {
    // Preload visible images
    this.preloadInitialImages();
  }

  trackByFn(index: number, item: ImageItem): string {
    return item.id;
  }

  onImageLoaded(imageId: string) {
    this.loadedImages.add(imageId);
    console.log(`Image ${imageId} loaded successfully`);
  }

  onImageError(imageId: string) {
    console.error(`Failed to load image ${imageId}`);
  }

  private preloadInitialImages() {
    // Preload first few images for better UX
    const initialImages = this.images.slice(0, 3);
    initialImages.forEach((image) => {
      const img = new Image();
      img.src = image.src;
    });
  }
}

interface ImageItem {
  id: string;
  src: string;
  placeholder: string;
  alt: string;
  title: string;
  description: string;
}
```

## 🛠️ **Image Optimization Utilities**

### **1. 📐 Image Resizing Service**

```typescript
// image-resize.service.ts
@Injectable({
  providedIn: "root",
})
export class ImageResizeService {
  resizeImage(
    file: File,
    maxWidth: number,
    maxHeight: number,
    quality = 0.8
  ): Promise<Blob> {
    return new Promise((resolve, reject) => {
      const canvas = document.createElement("canvas");
      const ctx = canvas.getContext("2d")!;
      const img = new Image();

      img.onload = () => {
        // Calculate new dimensions
        const { width, height } = this.calculateDimensions(
          img.width,
          img.height,
          maxWidth,
          maxHeight
        );

        canvas.width = width;
        canvas.height = height;

        // Draw and compress
        ctx.drawImage(img, 0, 0, width, height);

        canvas.toBlob(
          (blob) => {
            if (blob) {
              resolve(blob);
            } else {
              reject(new Error("Failed to resize image"));
            }
          },
          "image/jpeg",
          quality
        );
      };

      img.onerror = () => reject(new Error("Failed to load image"));
      img.src = URL.createObjectURL(file);
    });
  }

  generateThumbnail(file: File, size = 150): Promise<string> {
    return new Promise((resolve, reject) => {
      const canvas = document.createElement("canvas");
      const ctx = canvas.getContext("2d")!;
      const img = new Image();

      img.onload = () => {
        canvas.width = size;
        canvas.height = size;

        // Calculate crop area for square thumbnail
        const minDimension = Math.min(img.width, img.height);
        const sx = (img.width - minDimension) / 2;
        const sy = (img.height - minDimension) / 2;

        ctx.drawImage(
          img,
          sx,
          sy,
          minDimension,
          minDimension,
          0,
          0,
          size,
          size
        );

        resolve(canvas.toDataURL("image/jpeg", 0.8));
      };

      img.onerror = () => reject(new Error("Failed to generate thumbnail"));
      img.src = URL.createObjectURL(file);
    });
  }

  private calculateDimensions(
    originalWidth: number,
    originalHeight: number,
    maxWidth: number,
    maxHeight: number
  ): { width: number; height: number } {
    let { width, height } = { width: originalWidth, height: originalHeight };

    if (width > height) {
      if (width > maxWidth) {
        height = (height * maxWidth) / width;
        width = maxWidth;
      }
    } else {
      if (height > maxHeight) {
        width = (width * maxHeight) / height;
        height = maxHeight;
      }
    }

    return { width: Math.round(width), height: Math.round(height) };
  }

  convertToWebP(file: File, quality = 0.8): Promise<Blob> {
    return new Promise((resolve, reject) => {
      const canvas = document.createElement("canvas");
      const ctx = canvas.getContext("2d")!;
      const img = new Image();

      img.onload = () => {
        canvas.width = img.width;
        canvas.height = img.height;
        ctx.drawImage(img, 0, 0);

        canvas.toBlob(
          (blob) => {
            if (blob) {
              resolve(blob);
            } else {
              reject(new Error("Failed to convert to WebP"));
            }
          },
          "image/webp",
          quality
        );
      };

      img.onerror = () => reject(new Error("Failed to load image"));
      img.src = URL.createObjectURL(file);
    });
  }
}
```

## 🚨 **Performance Best Practices**

### **✅ Comprehensive Optimization Checklist**

1. **🎯 Choose the Right Format**

```typescript
// Format selection based on browser support
const getOptimalFormat = (): string => {
  if (supportsFormat("avif")) return "avif";
  if (supportsFormat("webp")) return "webp";
  return "jpeg";
};

const supportsFormat = (format: string): boolean => {
  const canvas = document.createElement("canvas");
  return canvas.toDataURL(`image/${format}`).startsWith(`data:image/${format}`);
};
```

2. **📏 Responsive Images Strategy**

```typescript
// Generate responsive image URLs
const generateResponsiveUrls = (baseUrl: string) => ({
  small: `${baseUrl}_480w.webp 480w`,
  medium: `${baseUrl}_768w.webp 768w`,
  large: `${baseUrl}_1200w.webp 1200w`,
  xlarge: `${baseUrl}_1920w.webp 1920w`,
});
```

3. **🔄 Preloading Strategy**

```typescript
// Smart preloading service
@Injectable()
export class ImagePreloadService {
  preloadCriticalImages(urls: string[]): void {
    urls.forEach((url) => {
      const link = document.createElement("link");
      link.rel = "preload";
      link.as = "image";
      link.href = url;
      document.head.appendChild(link);
    });
  }

  preloadOnIdle(urls: string[]): void {
    if ("requestIdleCallback" in window) {
      requestIdleCallback(() => this.preloadImages(urls));
    } else {
      setTimeout(() => this.preloadImages(urls), 100);
    }
  }
}
```

## 📊 **Angular Version Comparison**

| Feature                   | Angular 15            | Angular 17-19       |
| ------------------------- | --------------------- | ------------------- |
| **NgOptimizedImage**      | Basic support         | Enhanced features   |
| **Built-in Lazy Loading** | Manual implementation | Native support      |
| **Image Optimization**    | Third-party solutions | Built-in directives |
| **Performance**           | Custom optimization   | Framework-optimized |

## 🎯 **Key Takeaways**

### **Essential Optimization Techniques:**

1. **🖼️ Use NgOptimizedImage** for modern Angular applications
2. **🔄 Implement progressive loading** for better UX
3. **📱 Serve responsive images** based on device capabilities
4. **⚡ Lazy load images** below the fold
5. **📦 Choose modern formats** (WebP, AVIF) with fallbacks
6. **🎯 Preload critical images** for immediate visibility
7. **💾 Cache images effectively** to avoid re-downloads

### **Performance Metrics to Monitor:**

- **Largest Contentful Paint (LCP)**
- **First Contentful Paint (FCP)**
- **Cumulative Layout Shift (CLS)**
- **Total Blocking Time (TBT)**
- **Image load completion rate**

### **Common Pitfalls to Avoid:**

- Loading all images eagerly
- Not providing width/height attributes
- Using oversized images for small displays
- Ignoring modern image formats
- Not implementing proper error handling
- Missing accessibility considerations

Proper image optimization can dramatically improve your Angular application's performance and user experience! 🚀
