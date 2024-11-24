## Software  Versions

``` 
Angular CLI: 18.2.12

Node: 22.1.0

Package Manager: npm 10.7.0

OS: darwin arm64
```

### Step 1: Set Up Angular Applications

1. **Create Angular Applications**: Run the Angular CLI commands to generate your apps:
``` 
ng new host --routing --style=scss
ng new health --routing --style=scss
ng new retirement --routing --style=scss
```

### Step 2: Install Native Federation 
Run the following command in each of the projects. 
``` 
npm i @angular-architects/native-federation -D
```

### Step3: Execute automated setup for host
``` 
ng g @angular-architects/native-federation:init --port 4200 --type dynamic-host
```

This command creates the following files
1. federation.manifest.json
  ``` 
{  
  "mfe1": "http://localhost:4201/remoteEntry.json",  
  "mfe2": "http://localhost:4202/remoteEntry.json"  
}
```
2. federation.config.js
``` 
const { withNativeFederation, shareAll } = require('@angular-architects/native-federation/config');

module.exports = withNativeFederation({

  shared: {
    ...shareAll({ singleton: true, strictVersion: true, requiredVersion: 'auto' }),
  },

  skip: [
    'rxjs/ajax',
    'rxjs/fetch',
    'rxjs/testing',
    'rxjs/webSocket',
    // Add further packages you don't need at runtime
  ]

  // Please read our FAQ about sharing libs:
  // https://shorturl.at/jmzH0
  
});
```
3. tsconfig.federation.json
``` 
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "outDir": "./out-tsc/app",
    "types": []
  },
  "files": [
    "src/main.ts"
  ],
  "include": [
    "src/**/*.d.ts"
  ]
}
```

### Step 4: Execute setup for remotes
For remote 1
``` 
ng g @angular-architects/native-federation:init --port 4201 --type remote
```

For remote 2
``` 
ng g @angular-architects/native-federation:init --port 4202 --type remote
```

For remote n

``` 
ng g @angular-architects/native-federation:init --port n --type remote
```

The files created for the remotes are the same except for the federation.config.js. By default, the "name" will inherit the name of your project, I've changed the name to match the default naming scheme of federation for mfe'n' as your n applications.
``` 
const { withNativeFederation, shareAll } = require('@angular-architects/native-federation/config');  
  
module.exports = withNativeFederation({  
  
  name: 'mfe1',  
  
  exposes: {  
    './Component': './src/app/app.component.ts',  
  },  
  
  shared: {  
    ...shareAll({ singleton: true, strictVersion: true, requiredVersion: 'auto' }),  
  },  
  
  skip: [  
    'rxjs/ajax',  
    'rxjs/fetch',  
    'rxjs/testing',  
    'rxjs/webSocket',  
    // Add further packages you don't need at runtime  
  ]  
  
  // Please read our FAQ about sharing libs:  
  // https://shorturl.at/jmzH0  
});
```
### Step 5: Configure manifest for host
The manifest file contains the "links" to the remoteEntry files. This link is what you'll bind to later on when we route to the pages. 
``` 
{  
  "mfe1": "http://localhost:4201/remoteEntry.json",  
  "mfe2": "http://localhost:4202/remoteEntry.json"  
}
```

I'm assuming for now this name must match the exported name within the remote applications. 
Correction, it does not need to match... just the url

### Step 6: Configure router
app.routes.ts
``` 
import { Routes } from '@angular/router';  
import {loadRemoteModule} from '@angular-architects/native-federation';  
  
export const APP_ROUTES: Routes = [  
  {  
    path: 'mfe1',  
    loadComponent: () =>  
      loadRemoteModule('mfe1', './Component').then((m) => m.AppComponent),  
  },{  
    path: 'mfe2',  
    loadComponent: () =>  
      loadRemoteModule('mfe2', './Component').then((m) => m.AppComponent),  
  },  
];
```

### Step 7: add navigation to root app
``` 
<nav>  
  <ul>    
	<li><a href="#home">Home</a></li>  
    <li><a href="#about">About</a></li>  
    <li><a routerLink="/mfe1">retirement</a></li>  
    <li><a routerLink="/mfe2">health</a></li>  
  </ul></nav>  
<router-outlet></router-outlet>
```