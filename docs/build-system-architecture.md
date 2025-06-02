# Build System Architecture

This document provides detailed implementation guidance for Embedy's build system, including progressive loading, bundle management, polyfill strategies, and fallback mechanisms.

## Progressive Feature Loading

### Capability Detection System

The progressive loader analyzes device and network capabilities to determine optimal feature loading:

```typescript
// core/progressive-loader.ts
interface DeviceCapabilities {
  bandwidth: '2G' | '3G' | '4G' | 'wifi';
  memory: number; // GB
  touchDevice: boolean;
  screenSize: 'small' | 'medium' | 'large';
  hasModernBrowser: boolean;
  supportsWebComponents: boolean;
  supportsIntersectionObserver: boolean;
}

class ProgressiveFeatureLoader {
  private capabilities: DeviceCapabilities;
  private loadedFeatures = new Set<string>();
  
  constructor() {
    this.capabilities = this.detectCapabilities();
  }
  
  private detectCapabilities(): DeviceCapabilities {
    return {
      bandwidth: this.detectBandwidth(),
      memory: this.detectMemory(),
      touchDevice: 'ontouchstart' in window,
      screenSize: this.detectScreenSize(),
      hasModernBrowser: this.detectModernBrowser(),
      supportsWebComponents: 'customElements' in window,
      supportsIntersectionObserver: 'IntersectionObserver' in window
    };
  }
  
  private detectBandwidth(): '2G' | '3G' | '4G' | 'wifi' {
    const connection = (navigator as any).connection || (navigator as any).mozConnection;
    if (connection) {
      const effectiveType = connection.effectiveType;
      return effectiveType || '3G';
    }
    
    // Fallback: measure connection speed with a small test request
    return this.measureConnectionSpeed();
  }
  
  private async measureConnectionSpeed(): Promise<'2G' | '3G' | '4G' | 'wifi'> {
    const startTime = performance.now();
    try {
      await fetch('/ping.json?' + Math.random(), { cache: 'no-cache' });
      const duration = performance.now() - startTime;
      
      if (duration > 1000) return '2G';
      if (duration > 300) return '3G';
      if (duration > 100) return '4G';
      return 'wifi';
    } catch {
      return '3G'; // Conservative default
    }
  }
  
  private detectMemory(): number {
    const memory = (navigator as any).deviceMemory;
    return memory || 4; // Default to 4GB if unknown
  }
  
  async loadOptimalFeatureSet(): Promise<ComponentLibrary> {
    // Core features (always loaded - 18KB gzipped)
    const core = await import('./core/forms');
    
    // Progressive enhancement based on capabilities
    const features = await Promise.allSettled([
      this.loadAdvancedValidation(),
      this.loadMobileOptimizations(),
      this.loadVirtualScrolling(),
      this.loadRichFormatting(),
      this.loadAccessibilityFeatures()
    ]);
    
    return this.assembleLibrary(core, features.filter(this.isSuccessful));
  }
  
  private async loadAdvancedValidation(): Promise<any> {
    if (this.capabilities.hasModernBrowser && 
        this.capabilities.bandwidth !== '2G' &&
        this.capabilities.memory >= 2) {
      return import('./features/advanced-validation'); // +8KB
    }
    return null;
  }
  
  private async loadMobileOptimizations(): Promise<any> {
    if (this.capabilities.touchDevice) {
      return import('./features/mobile-optimizations'); // +5KB
    }
    return null;
  }
  
  private async loadVirtualScrolling(): Promise<any> {
    if (this.capabilities.supportsIntersectionObserver &&
        this.capabilities.memory >= 4) {
      return import('./features/virtual-scrolling'); // +6KB
    }
    return null;
  }
  
  private async loadRichFormatting(): Promise<any> {
    if (this.capabilities.bandwidth === '4G' || this.capabilities.bandwidth === 'wifi') {
      return import('./features/rich-formatting'); // +12KB
    }
    return null;
  }
  
  private isSuccessful(result: PromiseSettledResult<any>): result is PromiseFulfilledResult<any> {
    return result.status === 'fulfilled' && result.value !== null;
  }
}
```

### Bundle Splitting Strategy

The build system uses manual chunk splitting for optimal caching and loading:

```typescript
// vite.config.ts - Enhanced configuration
export default defineConfig({
  build: {
    rollupOptions: {
      input: {
        // Core entry points
        'core': './src/core/index.ts',
        'web-components': './src/web-components/index.ts',
        'react': './src/react/index.ts',
        'iframe': './src/iframe/index.ts'
      },
      output: {
        // Manual chunk splitting for optimal caching
        manualChunks: (id) => {
          // Vendor libraries (rarely change, long cache)
          if (id.includes('node_modules')) {
            if (id.includes('lit')) return 'vendor-lit';
            if (id.includes('react')) return 'vendor-react';
            if (id.includes('zod')) return 'vendor-validation';
            if (id.includes('postmate')) return 'vendor-iframe';
            return 'vendor-misc';
          }
          
          // Feature chunks (loaded conditionally)
          if (id.includes('src/features/')) {
            const feature = id.split('src/features/')[1].split('/')[0];
            return `feature-${feature}`;
          }
          
          // Component chunks (loaded on demand)
          if (id.includes('src/components/')) {
            const component = id.split('src/components/')[1].split('/')[0];
            return `component-${component}`;
          }
          
          // Theme chunks (shared across components)
          if (id.includes('src/themes/')) {
            return 'themes';
          }
          
          // Core functionality (always loaded)
          if (id.includes('src/core/')) {
            return 'core';
          }
        },
        chunkFileNames: (chunkInfo) => {
          // Predictable chunk naming for caching strategies
          const name = chunkInfo.name;
          if (name.startsWith('vendor-')) {
            return `vendors/[name].[hash].js`;
          } else if (name.startsWith('feature-')) {
            return `features/[name].[hash].js`;
          } else if (name.startsWith('component-')) {
            return `components/[name].[hash].js`;
          }
          return `chunks/[name].[hash].js`;
        }
      }
    }
  }
});

// Dynamic chunk loading with caching
class ChunkManager {
  private loadedChunks = new Set<string>();
  private chunkPromises = new Map<string, Promise<any>>();
  private chunkCache = new Map<string, any>();
  
  async loadChunk(chunkName: string): Promise<any> {
    // Check cache first
    if (this.chunkCache.has(chunkName)) {
      return this.chunkCache.get(chunkName);
    }
    
    if (this.loadedChunks.has(chunkName)) {
      return; // Already loaded
    }
    
    if (this.chunkPromises.has(chunkName)) {
      return this.chunkPromises.get(chunkName); // Loading in progress
    }
    
    const promise = this.loadChunkWithFallback(chunkName);
    this.chunkPromises.set(chunkName, promise);
    
    try {
      const result = await promise;
      this.loadedChunks.add(chunkName);
      this.chunkCache.set(chunkName, result);
      return result;
    } catch (error) {
      this.chunkPromises.delete(chunkName);
      throw error;
    }
  }
  
  private async loadChunkWithFallback(chunkName: string): Promise<any> {
    const chunkPaths = [
      `/chunks/${chunkName}.js`,
      `/fallbacks/${chunkName}.js`,
      `/core/minimal-${chunkName}.js`
    ];
    
    for (const path of chunkPaths) {
      try {
        return await import(path);
      } catch (error) {
        console.warn(`Failed to load chunk from ${path}, trying next fallback`);
      }
    }
    
    throw new Error(`Failed to load chunk: ${chunkName}`);
  }
}
```

## Polyfill Management

### Feature Detection and Conditional Loading

```typescript
// core/polyfill-loader.ts
interface PolyfillConfig {
  features: string[];
  loadStrategy: 'eager' | 'lazy' | 'on-demand';
  fallbackTimeout: number;
}

class PolyfillLoader {
  private static readonly FEATURE_TESTS = {
    'custom-elements': () => 'customElements' in window,
    'shadow-dom': () => 'attachShadow' in Element.prototype,
    'es6-modules': () => 'noModule' in HTMLScriptElement.prototype,
    'fetch': () => 'fetch' in window,
    'promises': () => 'Promise' in window,
    'web-crypto': () => 'crypto' in window && 'subtle' in crypto,
    'intersection-observer': () => 'IntersectionObserver' in window,
    'resize-observer': () => 'ResizeObserver' in window,
    'css-custom-properties': () => window.CSS && CSS.supports('color', 'var(--test)'),
    'css-containment': () => window.CSS && CSS.supports('contain', 'layout'),
    'es2020-features': () => {
      try {
        // Test for optional chaining and nullish coalescing
        eval('const test = {}; test?.prop ?? "default"');
        return true;
      } catch {
        return false;
      }
    }
  };
  
  private static readonly POLYFILL_MAP = {
    'custom-elements': () => import('@webcomponents/webcomponentsjs/custom-elements-es5-adapter.js'),
    'shadow-dom': () => import('@webcomponents/webcomponentsjs/webcomponents-bundle.js'),
    'fetch': () => import('whatwg-fetch'),
    'promises': () => import('es6-promise/auto'),
    'web-crypto': () => import('webcrypto-shim'),
    'intersection-observer': () => import('intersection-observer'),
    'resize-observer': () => import('resize-observer-polyfill'),
    'css-custom-properties': () => import('css-vars-ponyfill'),
    'es2020-features': () => import('core-js/stable'),
    'es6-modules': () => this.loadNoModuleFallback()
  };
  
  static async loadRequiredPolyfills(config: PolyfillConfig = {
    features: ['custom-elements', 'shadow-dom', 'fetch', 'promises'],
    loadStrategy: 'eager',
    fallbackTimeout: 5000
  }): Promise<void> {
    const missingFeatures = this.detectMissingFeatures(config.features);
    
    if (missingFeatures.length === 0) {
      return; // All features are natively supported
    }
    
    console.info(`Loading polyfills for: ${missingFeatures.join(', ')}`);
    
    switch (config.loadStrategy) {
      case 'eager':
        await this.loadPolyfillsEager(missingFeatures, config.fallbackTimeout);
        break;
      case 'lazy':
        await this.loadPolyfillsLazy(missingFeatures);
        break;
      case 'on-demand':
        this.setupOnDemandPolyfills(missingFeatures);
        break;
    }
  }
  
  private static detectMissingFeatures(requestedFeatures: string[]): string[] {
    return requestedFeatures.filter(feature => {
      const test = this.FEATURE_TESTS[feature];
      if (!test) {
        console.warn(`Unknown feature test: ${feature}`);
        return false;
      }
      return !test();
    });
  }
  
  private static async loadPolyfillsEager(features: string[], timeout: number): Promise<void> {
    const polyfillPromises = features.map(feature => {
      const loader = this.POLYFILL_MAP[feature];
      if (!loader) {
        console.warn(`No polyfill available for: ${feature}`);
        return Promise.resolve();
      }
      
      return Promise.race([
        loader(),
        new Promise((_, reject) => 
          setTimeout(() => reject(new Error(`Polyfill timeout: ${feature}`)), timeout)
        )
      ]).catch(error => {
        console.error(`Failed to load polyfill for ${feature}:`, error);
        // Continue without this polyfill
      });
    });
    
    await Promise.allSettled(polyfillPromises);
  }
  
  private static async loadPolyfillsLazy(features: string[]): Promise<void> {
    // Load polyfills when the page is idle
    if ('requestIdleCallback' in window) {
      return new Promise(resolve => {
        requestIdleCallback(async () => {
          await this.loadPolyfillsEager(features, 10000);
          resolve();
        });
      });
    } else {
      // Fallback for browsers without requestIdleCallback
      await new Promise(resolve => setTimeout(resolve, 100));
      await this.loadPolyfillsEager(features, 10000);
    }
  }
  
  private static setupOnDemandPolyfills(features: string[]): void {
    // Setup lazy loading for when features are actually needed
    features.forEach(feature => {
      this.createFeatureProxy(feature);
    });
  }
  
  private static createFeatureProxy(feature: string): void {
    switch (feature) {
      case 'custom-elements':
        if (!('customElements' in window)) {
          // Create a proxy that loads the polyfill when accessed
          (window as any).customElements = new Proxy({}, {
            get: async (target, prop) => {
              await this.POLYFILL_MAP[feature]();
              return window.customElements[prop];
            }
          });
        }
        break;
      
      case 'intersection-observer':
        if (!('IntersectionObserver' in window)) {
          (window as any).IntersectionObserver = class {
            constructor(...args: any[]) {
              return this.POLYFILL_MAP[feature]().then(module => {
                return new module.default(...args);
              });
            }
          };
        }
        break;
    }
  }
}
```

## Fallback Management

### Multi-Tier Fallback Strategy

```typescript
// core/fallback-manager.ts
interface FallbackConfig {
  retryAttempts: number;
  retryDelay: number;
  fallbackUrls: string[];
  gracefulDegradation: boolean;
}

class FallbackManager {
  private static readonly DEFAULT_CONFIG: FallbackConfig = {
    retryAttempts: 3,
    retryDelay: 1000,
    fallbackUrls: [
      'https://cdn.embedy.com/v1/',
      'https://backup-cdn.embedy.com/v1/',
      '/vendor/embedy/'
    ],
    gracefulDegradation: true
  };
  
  static async loadWithFallback<T>(
    primaryImport: () => Promise<T>,
    fallbackStrategies: Array<() => Promise<T>>,
    config: Partial<FallbackConfig> = {}
  ): Promise<T> {
    const fullConfig = { ...this.DEFAULT_CONFIG, ...config };
    
    // Try primary import with retries
    try {
      return await this.retryImport(primaryImport, fullConfig.retryAttempts, fullConfig.retryDelay);
    } catch (primaryError) {
      console.warn('Primary import failed:', primaryError);
    }
    
    // Try fallback strategies in order
    for (let i = 0; i < fallbackStrategies.length; i++) {
      try {
        console.info(`Attempting fallback strategy ${i + 1}/${fallbackStrategies.length}`);
        return await this.retryImport(fallbackStrategies[i], 2, fullConfig.retryDelay);
      } catch (fallbackError) {
        console.warn(`Fallback strategy ${i + 1} failed:`, fallbackError);
      }
    }
    
    // All strategies failed
    if (fullConfig.gracefulDegradation) {
      return this.createGracefulFallback();
    } else {
      throw new Error('All import strategies failed');
    }
  }
  
  private static async retryImport<T>(
    importFn: () => Promise<T>,
    attempts: number,
    delay: number
  ): Promise<T> {
    let lastError: Error;
    
    for (let attempt = 1; attempt <= attempts; attempt++) {
      try {
        return await importFn();
      } catch (error) {
        lastError = error as Error;
        
        if (attempt < attempts) {
          console.warn(`Import attempt ${attempt} failed, retrying in ${delay}ms`);
          await new Promise(resolve => setTimeout(resolve, delay * attempt));
        }
      }
    }
    
    throw lastError!;
  }
  
  private static createGracefulFallback<T>(): T {
    // Return a minimal implementation that prevents crashes
    return {
      initialize: () => Promise.resolve(),
      render: () => '<div class="embedy-fallback">Component temporarily unavailable</div>',
      destroy: () => {},
      updateTheme: () => Promise.resolve(),
      validate: () => Promise.resolve({ valid: true, errors: [] }),
      submit: () => Promise.reject(new Error('Feature unavailable'))
    } as T;
  }
  
  // Specific fallback strategies for different types of imports
  static createComponentFallbacks(componentName: string): Array<() => Promise<any>> {
    return [
      // Strategy 1: Try alternative CDN
      () => import(`https://backup-cdn.embedy.com/components/${componentName}.js`),
      
      // Strategy 2: Try local bundle
      () => import(`/vendor/embedy/components/${componentName}.js`),
      
      // Strategy 3: Load from alternative build
      () => import(`/dist/components/${componentName}.legacy.js`),
      
      // Strategy 4: Load basic implementation
      () => import(`/fallbacks/basic-${componentName}.js`)
    ];
  }
  
  static createFeatureFallbacks(featureName: string): Array<() => Promise<any>> {
    return [
      // Strategy 1: Try reduced feature set
      () => import(`/features/${featureName}-lite.js`),
      
      // Strategy 2: Load polyfill version
      () => import(`/polyfills/${featureName}-polyfill.js`),
      
      // Strategy 3: Return no-op implementation
      () => Promise.resolve({ default: this.createNoOpFeature(featureName) })
    ];
  }
  
  private static createNoOpFeature(featureName: string) {
    console.warn(`Using no-op implementation for feature: ${featureName}`);
    return {
      initialize: () => Promise.resolve(),
      enable: () => {},
      disable: () => {},
      isSupported: () => false
    };
  }
}

// Network-aware loading
class NetworkAwareLoader {
  private isOnline = navigator.onLine;
  private connectionQuality = this.detectConnectionQuality();
  
  constructor() {
    window.addEventListener('online', () => this.isOnline = true);
    window.addEventListener('offline', () => this.isOnline = false);
  }
  
  async loadComponent(componentName: string): Promise<any> {
    if (!this.isOnline) {
      // Try to load from cache or local storage
      return this.loadFromCache(componentName);
    }
    
    if (this.connectionQuality === 'slow') {
      // Load minimal version for slow connections
      return this.loadMinimalVersion(componentName);
    }
    
    // Normal loading with fallbacks
    return FallbackManager.loadWithFallback(
      () => import(`./components/${componentName}`),
      FallbackManager.createComponentFallbacks(componentName)
    );
  }
  
  private async loadFromCache(componentName: string): Promise<any> {
    const cached = localStorage.getItem(`embedy-component-${componentName}`);
    if (cached) {
      try {
        return JSON.parse(cached);
      } catch (error) {
        console.warn('Failed to parse cached component:', error);
      }
    }
    
    // Return offline fallback
    return this.createOfflineFallback(componentName);
  }
  
  private createOfflineFallback(componentName: string) {
    return {
      render: () => `<div class="embedy-offline">
        <h3>${componentName}</h3>
        <p>This component is temporarily unavailable while offline.</p>
        <button onclick="window.location.reload()">Retry when online</button>
      </div>`
    };
  }
  
  private detectConnectionQuality(): 'fast' | 'slow' {
    const connection = (navigator as any).connection;
    if (connection) {
      return connection.effectiveType === '2G' || connection.effectiveType === 'slow-2g' ? 'slow' : 'fast';
    }
    return 'fast';
  }
}
```

## Usage Examples

### Application Bootstrap

```typescript
// Application initialization with progressive loading
async function initializeEmbedy() {
  try {
    // Load required polyfills first
    await PolyfillLoader.loadRequiredPolyfills({
      features: ['custom-elements', 'shadow-dom', 'fetch', 'intersection-observer'],
      loadStrategy: 'eager',
      fallbackTimeout: 5000
    });
    
    // Initialize progressive loader
    const progressiveLoader = new ProgressiveFeatureLoader();
    const componentLibrary = await progressiveLoader.loadOptimalFeatureSet();
    
    // Initialize core with loaded features
    const { EmbedyCore } = await import('./core/embedy-core');
    await EmbedyCore.initialize(componentLibrary);
    
    console.log('Embedy initialized successfully');
    
  } catch (error) {
    console.error('Failed to initialize Embedy:', error);
    
    // Load minimal fallback implementation
    const fallbackLibrary = await import('./fallbacks/basic-forms');
    await fallbackLibrary.initialize();
  }
}

// Component loading with fallbacks
class ComponentLoader {
  private fallbackManager = new FallbackManager();
  private networkLoader = new NetworkAwareLoader();
  
  async loadInvoiceForm(): Promise<any> {
    return FallbackManager.loadWithFallback(
      // Primary import
      () => import('./components/invoice-form'),
      
      // Fallback strategies
      FallbackManager.createComponentFallbacks('invoice-form'),
      
      // Configuration
      {
        retryAttempts: 2,
        gracefulDegradation: true
      }
    );
  }
  
  async loadAdvancedValidation(): Promise<any> {
    return FallbackManager.loadWithFallback(
      () => import('./features/advanced-validation'),
      FallbackManager.createFeatureFallbacks('advanced-validation'),
      {
        gracefulDegradation: true // Can work without this feature
      }
    );
  }
}
```

This build system architecture ensures:

1. **Optimal Loading**: Components and features are loaded based on actual device capabilities
2. **Efficient Caching**: Strategic bundle splitting for long-term caching
3. **Graceful Degradation**: Multiple fallback strategies prevent failures
4. **Network Awareness**: Adapts to connection quality and offline scenarios
5. **Progressive Enhancement**: Core functionality always available, advanced features when supported