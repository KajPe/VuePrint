# VuePrint
Print plugin (component) for Vue
  
  
## How it works
- IFrame support only (no popup)
- The printout (what gets inside the iframe) is either built in code or can be downloaded with axios
- If multiple vue-print components are placed then each component has an own iframe. This is by design.  
  The iframe is however emptied after a print, so should not cause much problems.
- State is communicated back to parent via events
- The component is listening on then click event, the visual part can be a button, image, icon, ...
- Tested with Chrome 87 & FireFox 84

## Usage
In your parent component import the plugin in the **script** section.

```javascript
import VuePrint from '../VuePrint'
```

Add **VuePrint** to the components list
```javascript
export default {
    components: {
        VuePrint
    }
}
```

You can now add **vue-print** to your template. Clicking the **print** text will start the print process.
```javascript
  <vue-print :callback="printcallback">Print</vue-print>
```

You can simply configure anything you want to visualize the vue-print.  
For example here is Vuetify's components v-btn and v-icon used.
```javascript
  <vue-print :callback="printcallback">
    <v-btn color="primary">
      Print
      <v-icon>fa-print</v-icon>
    </v-btn>
  </vue-print>
```

### Props
- callback [mandatory]
  This callback will return the printout - either a string or promise - for the print component.

  **String**  
  Content can be built in code and simply returned as a string
  ```javascript
    <vue-print :callback="printcallback">Print</vue-print>
  
    printcallback() {
      return '<html><body>Hello</body></html>'
    }
  ```

  **Promise**  
  Content can be retrieved from an url with axios (anything which returns a promise should work)
  ```javascript
    <vue-print :callback="printcallback">Print</vue-print>
  
    printcallback() {
      return axios.get(url)
    }
  ```

- timeout [optional, default 5000ms]  
  Currently this is the timeout for the **prosessing** of the main content (which was retrieved with the callback props).  
  So this includes anything loaded **after** the main content (images, javascript, ...)
  
  ```javascript
    // Set timeout to 20 seconds
    <vue-print :callback="printcallback" :timeout=20000>Print</vue-print>
  ```

### Events
- onLoading(state)  [optional]
  This will emit a state of true when the print process starts (clicked), it will go false just before:
  - the printer dialog opens
  - download fails for some reason
  
  It can simply be used to show an loading dialog to the user.
  
  ```javascript
    <vue-print :callback="printcallback" @onLoading="onLoading">Print</vue-print>

    onLoading(state) {
      console.log("Loading state is ", state)
    }
  ```

- onAfterPrint  [optional]  
  This will trigger after the printer dialog has opened.  
  This dont mean the document has been printed; user can still cancel the print.
   
  ```javascript
    <vue-print :callback="printcallback" @onAfterPrint="onAfterPrint">Print</vue-print>

    onAfterPrint() {
      console.log("Print done")
    }
  ```

- onTimeoutError  [optional]  
  Triggered when timeout has occured (see props timeout).  
  Can be used to show an error dialog.
   
  ```javascript
    <vue-print :callback="printcallback" @onTimeoutError="onTimeoutError">Print</vue-print>

    onTimeoutError() {
      console.log("Timeout !")
    }
  ```

- onLoadError  [optional]  
  Triggered when loading the content fails for some reason (content retrieved via callback).  
  Can be used to show an error dialog.
   
  ```javascript
    <vue-print :callback="printcallback" @onLoadError="onLoadError">Print</vue-print>

    onLoadError() {
      console.log("Error retrieving content !")
    }
  ```

