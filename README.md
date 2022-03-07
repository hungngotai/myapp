# Language support <!-- omit in toc -->

- [1. Overview](#1-overview)
- [2. Specs](#2-specs)
  - [2.1. Language text definitions](#21-language-text-definitions)
  - [2.2. LanguageProvider and examples](#22-languageprovider-and-examples)
- [3. TODO](#3-todo)

# 1. Overview

To switch display of texts in different languages, use a language switch mechanism.

![button-example](/uploads/9f4fcdddb6eb349073f34a4c7f8f70b3/button-example.png)

# 2. Specs
          
## 2.1. Language text definitions 

- So far, we define Japanese and English.
  - Can support Vietnamese:)
- General words should be defined in `utils/LanguageSupport/<lang>.js`.
  - Each word's id has prefix `@GEN-`.
- Each component has `language` directory which contains each text definitions (i.e. 'en.js', 'ja.js', etc.).
  - Should has unique id prefix beginning with *@* (e.g. `@PGB-select-all-groups`).
- A script will concatenate all language definitions before easier library is imported.
  - It finds language definition in `language` directories.

```
repo-root
├── easier 
│   ├── plot
│   │   ├── EupGroupBarPlot
│   │   └── EupScatterPlot
│   ├── ui
│   │   ├── EuiSpecialButton
│   │   ├── EuiSelectableTable
│   │   └── EuiTable
│   │       ├── README.md
│   │       ├── language   <============== Language text definitions of the component
│   │       │   ├── ja.js
│   │       │   └── en.js
│   │       ├── filterLogic.js
│   │       ├── filterLogic.test.js 
│   │       ├── index.js  # interface
│   │       ├── sortLogic.js # some internal logic
│   │       ├── sortLogic.test.js # unit test with jest
│   │       ├── table.stories.js # for storybook.js
│   │       └── table.test.js # react-jest
│   └── utils
│       └── LanguageSupport
│           ├── README.md
│           ├── LanguageProvider.js
│           ├── en.js   <============== General words
│           └── ja.js   <==============           definitions
```


## 2.2. LanguageProvider and examples

Following are example and definition of `LanguageProvider`.



- General Japanese 
    ```js
    /*
        ja.js

        Copyright 2021. ADVANTEST Corporation. All rights reserved.
    */

    export default {
        '@GEN-yes': 'はい',
        '@GEN-no': 'いいえ',
        '@GEN-all': '全て',
    };
    ```

- Button's Japanese
    ```js
    /*
        Button/language/ja.js

        Copyright 2021. ADVANTEST Corporation. All rights reserved.
    */

    export default {
        '@BTN-label': '{what}なボタン',
    };
    ```

- General English
    ```js
    /*
        en.js

        Copyright 2021. ADVANTEST Corporation. All rights reserved.
    */

    export default {
        '@GEN-yes': 'Yes!',
        '@GEN-no': 'No...',
        '@GEN-all': 'All',
    };
    ```

- Button's English
    ```js
    /*
        Button/language/en.js

        Copyright 2021. ADVANTEST Corporation. All rights reserved.
    */

    export default {
        '@BTN-button-label': '{what} Button',
    };
    ```

- Language provider  
  This code is based on our program currently working.
    ```js

    /*
        LanguageProvider.js

        Copyright 2021. ADVANTEST Corporation. All rights reserved.
    */

    // Definition of language provider
    import React, { useReducer } from 'react';

    // General words
    import message_jajp_general from './ja'; 
    import message_enus_general from './en'; 

    // From all components, made by script 
    import message_jajp_components from './ja-components'; 
    import message_enus_components from './en-components'; 

    // concatenate
    const message_jajp = Object.assign({}, message_jajp_general, message_jajp_components);
    const message_enus = Object.assign({}, message_enus_general, message_enus_components);

    /**Convert JavaScript Object to Map for fast resolve of key.
     * 
     * @param {Object} obj 
     * @returns Map
     */
    function obj2Map(obj){
        return new Map(Object.entries(obj))
    };

    /**Language map definitions.
     * - ja: Japanese
     * - en: English
    */
    export const messages = {
        ja: message_jajp,
        en: message_enus,
    };

    const messageMaps = {
        ja: obj2Map(message_jajp),
        en: obj2Map(message_enus),
    };

    const languageState = messages['en'];
    const languageContext = React.createContext(languageState);

    /**Context provider.
     * 
     * Examples:
     *  const App = () => {
     *      const language = getUserSelectedLanguage();
     *  
     *      return (
     *          <LanguageProvider 
     *              language={language}
     *          >
     *              <MyButton />
     *          </LanguageProvider>
     *      );
     *  };
    */
    export const LanguageProvider = (props) =>{
        const {language} = props;
        const selectedLanguage = language in messageMaps?messageMaps[language]:messageMaps['en'];

        return (
            <languageContext.Provider value={selectedLanguage}>
                {props.children}
            </languageContext.Provider>
        );
    }


    /**Get message text in selected language.
     * 
     * Examples:
     *  import { useFormattedMessage } from 'path/to/LanguageProvider';
     * 
     *  const MyButton = () => {
     *      const formattedMessage = useFormattedMessage();
     *      const foo = 'Good';
     * 
     *      return (
     *          <Button 
     *              label={formattedMessage('@BTN-button-label',values={'what':foo})}
     *          />
     *      );
     *  };
    */
    export const useFormattedMessage = () => {
        const langMessages = React.useContext(languageContext);

        return (id, values) => {
            // Get message from Map
            let message = langMessages.get(id);

            // Return just key if undefined key
            if(!message){
                return id;
            }

            // Replace placeholders 
            if(values){
                for(const [key,value] of Object.entries(values)){
                    const pattern = new RegExp('\\{\\s*' + key + '\\s*\\}','g');
                    message = message.replace(pattern,value);
                }
            }
            
            return message;
        };
    };

    ```


# 3. TODO

- Make integration script
- Storybook support?
- Add info into the document of *how-to-make-new-component*.

# EuiGroupTable <!-- omit in toc -->


- [1. Overview](#1-overview)
  - [1.1. Concerns of other library](#11-concerns-of-other-library)
- [2. Functional specs](#2-functional-specs)
  - [2.1. General specs](#21-general-specs)
  - [2.2. Tool bar](#22-tool-bar)
  - [2.3. Header row](#23-header-row)
  - [2.4. Data rows](#24-data-rows)
- [3. Data & code examples](#3-data--code-examples)

# 1. Overview

- An UI component of table view to display group data.
- The table is to display and control same data as EupGroupBarPlot.

## 1.1. Concerns of other library

- OSS table components wastes spaces.
- Users want to use table data in other documentations, so data export is desired.
- Because of division structure, scrolling is complicated. 
- Simple horizontal (column direction) scroll.
- Freeze header and key parts while scrolling.


# 2. Functional specs

![table-overview](/uploads/5c28aec6479bb4e0fb43eaa193446a09/table-overview.png)


## 2.1. General specs
- Show scrollbar when needed.
- Scroll freeze
  - Header rows.
  - Value columns  
  ![horizontal-scroll](/uploads/afd0a87bd6db55b6cf65021a132b7ca6/horizontal-scroll.png)
- Colors
  - Follow coloring of [Table of material ui](https://mui.com/components/tables/#main-content).
- Events
  - `onUpdateData(newData)`
    - To notify change of data to other components such like EupGroupBarPlot.
    - Invoked when;
      - change sort condition
      - change check/uncheck rows.
      - change check/uncheck value columns.
  

## 2.2. Tool bar

![toolbar](/uploads/572735dd1d946cea2ae2624cef24547a/toolbar.png)

- Title
  - String from `config.title`
  - Truncate when component width is small (e.g. 'long long title' ==> 'long long...').
- Check condition
  - Show `[<checked-row-count>/<row-count>]`
- TBD: ~~Value column scroll button~~
  - ~~Move value columns when there are many value columns and not all value columns can be shown at once.~~
- More button
  - *Column selection*
    - *Uncheck all columns*
    - *Check all columns*
    - ~~*Clear filters*~~
  - *Save as CSV file*
  - *Value display mode* (Save cookie which affects all EuiGroupTable)
    - *Show number*
    - *Show percentage* (percentge = `(groups[i].value[j] / groups[i].total) * 100`, to two decimal places)
    - *Show number and percentage* (default)  
      ![value-cell](/uploads/5ac74709b6da97b6e6efd0dc0aec484c/value-cell.png)

## 2.3. Header row

![header](/uploads/d94824403d5d211db19611e9d0588891/header.png)

- Checkbox: three state
  - Unchecked = all rows unchecked
  - Checked = all rows checked
  - Indeterminate = at least one row checked and at least one row unchecked
- Index column: empty
- Key columns
  - Display `data.keys.name`
- Total: fixed to *Total*
- Value columns
  - Display `data.barStacks.name`
  

### 2.3.1. Header cell

![header-column](/uploads/1729ec3e6a4be6ee2df03bf08be3d6aa/header-column.png)

- Checkbox
  - Shown only on value column
  - Check state equals to `data.barStacks[i].visible`.
- Name
  - Show `data.barStacks.name`, `data.keys.name`, or 'Total'
  - If it is **a value column**, text color is `data.barStacks[i].color`.
    - If `data.barStacks[i].color` is null or undefined, use default color.
- Sort button
  - change icon based on `data.misc.sort`


## 2.4. Data rows

![data-row](/uploads/8de10acdf78af02aa1b86df57757f736/data-row.png)

- Checkbox
  - Checked when `data.groups[i].visible` is true.
- Index column
  - Row's index in `data.groups` + 1
- Key columns
  - `data.groups[i].key-values[col-index]`
  - Text color = `data.groups[i].color`
- Total column
  - `data.groups[i].total`
- Value columns
  - `data.groups[i].bar-values[col-index]`
  - Three mode based on `Value display mode` of tool bar selection

# 3. Data & code examples

Structure of `data` is same as EupHorizontalBarPlot.

```
config = {
  'title': 'xyz data',
},

data= {
  "groups": [
    {
      "total": 255, // null or number
      "barValues": [10, 20, 30], // array of each stack
      "keyValues": ['x','A2', 3], // array of each keys
      "visible": true, // boolean
      "color": "#3f1891"
    },
    {
      "total": 200,
      "barValues": [10, 20, 30], // array of each stack
      "keyValues": ['y','A1', 10], // array of each keys
      "visible": false
      "color": "#901789"
    },
    {
      "total": 100,
      "barValues": [50, 10, 3], // array of each stack
      "keyValues": ['x','A1', 1], // array of each keys
      "visible": false
      "color": "#123456"
    }
  ],

  "barStacks": [
    {
      "name": "Bin1", // string
      "color": "#FF0000", // hex
      "visible": true, // for future enhancement
    },
    {
      "name": "Bin2",
      "color": "#00FF00",
      "visible": true,
    },
    {
      "name": "Bin3",
      "color": "#0000FF",
      "visible": true,
    },
  ],

  "keys": [
    {
      "name": "keyA",
    },
    {
      "name": "keyB",
    },
    {
      "name": "keyC",
    },
  ],

  "sort":{
    "by": "columnKey", // key name, value name, or total
    "order": "descending"
  }
}

function updateGroupBarPlot(newData){
  // update associated EupGroupBarPlot
}

<EuiGroupTable 
  config={config} 
  data={data} 
  onUpdateData={updateGroupBarPlot}
/>
```
