<p align="center"><br><img src="https://user-images.githubusercontent.com/236501/85893648-1c92e880-b7a8-11ea-926d-95355b8175c7.png" width="128" height="128" /></p>
<h2 align="center">IONIC/REACT USAGE DOCUMENTATION</h2>
<p align="center"><strong><code>@asello/capacitor-sqlite@latest</code></strong></p>
<p align="center">
  In Ionic/React Applications, the <code>@asello/capacitor-sqlite@latest</code> can be accessed through a Singleton React Hook initialized in the <code>App.tsx</code> component</p>
<br>

## React SQLite Hook

- [`React SQLite Hook Definition`](#react-sqlite-hook-definition)
- [`React SQLite Hook Declaration for platforms other than Web`](#react-sqlite-hook-declaration-for-platforms-other-than-Web)
- [`React SQLite Hook Declaration for platforms including Web`](#react-sqlite-hook-declaration-for-platforms-including-Web)
- [`React SQLite Hook Use in Components`](#react-sqlite-hook-use-in-components)

### React SQLite Hook Definition

A react hook specific to `@asello/capacitor-sqlite` plugin has been developed to access the plugin API

- [react-sqlite-hook](https://github.com/jepiqueau/react-sqlite-hook/blob/master/README.md)

To install it in your Ionic/React App

 - `for Native Apps`

```bash
    npm i --save-dev @asello/capacitor-sqlite@latest
    npm i --save-dev react-sqlite-hook@latest
```

 - `for Web Browser` 
```bash
    npm i --save-dev jeep-sqlite@latest
```


### React SQLite Hook Declaration for platforms other than Web

To use the `react-sqlite-hook`as a singleton hook, the declaration must be done in the `App.tsx` file of your application

```ts
...
import { useSQLite } from 'react-sqlite-hook';
...
// Singleton SQLite Hook
export let sqlite: any;
// Existing Connections Store
export let existingConn: any;

const App: React.FC = () => {
  const [existConn, setExistConn] = useState(false);
  existingConn = {existConn: existConn, setExistConn: setExistConn};
  const {echo, getPlatform, createConnection, closeConnection,
         retrieveConnection, retrieveAllConnections, closeAllConnections,
         addUpgradeStatement, importFromJson, isJsonValid, copyFromAssets,
         isAvailable} = useSQLite();
  sqlite = {echo: echo, getPlatform: getPlatform,
            createConnection: createConnection,
            closeConnection: closeConnection,
            retrieveConnection: retrieveConnection,
            retrieveAllConnections: retrieveAllConnections,
            closeAllConnections: closeAllConnections,
            addUpgradeStatement: addUpgradeStatement,
            importFromJson: importFromJson,
            isJsonValid: isJsonValid,
            copyFromAssets: copyFromAssets,
            isAvailable:isAvailable};
...
  return (
  <IonApp>
...
  </IonApp>
  )
};

export default App;
```

Now the Singleton SQLite Hook `sqlite`and Existing Connections Store `existingConn` can use in other components

### React SQLite Hook Declaration for platforms including Web
As for the Web platform, the `jeep-sqlite` Stencil component is used and requires the DOM it is then defined and initialized in the `index.tsx` file.

```ts
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import * as serviceWorker from './serviceWorker';
import { defineCustomElements as jeepSqlite, applyPolyfills, JSX as LocalJSX  } from "jeep-sqlite/loader";
import { HTMLAttributes } from 'react';
import { Capacitor } from '@capacitor/core';
import { CapacitorSQLite, SQLiteConnection, SQLiteDBConnection } from '@asello/capacitor-sqlite';

type StencilToReact<T> = {
  [P in keyof T]?: T[P] & Omit<HTMLAttributes<Element>, 'className'> & {
    class?: string;
  };
} ;

declare global {
  export namespace JSX {
    interface IntrinsicElements extends StencilToReact<LocalJSX.IntrinsicElements> {
    }
  }
}

applyPolyfills().then(() => {
  jeepSqlite(window);
});
window.addEventListener('DOMContentLoaded', async () => {
  const platform = Capacitor.getPlatform();
  const sqlite: SQLiteConnection = new SQLiteConnection(CapacitorSQLite)
  try {
    if(platform === "web") {
      const jeepEl = document.createElement("jeep-sqlite");
      document.body.appendChild(jeepEl);
      await customElements.whenDefined('jeep-sqlite');
      await sqlite.initWebStore();
    }
    const ret = await sqlite.checkConnectionsConsistency();
    const isConn = (await sqlite.isConnection("db_issue9")).result;
    /* the code below is an example if you wish to set-up the schema of your database */
    var db: SQLiteDBConnection
    if (ret.result && isConn) {
      db = await sqlite.retrieveConnection("db_issue9");
    } else {
      db = await sqlite.createConnection("db_issue9", false, "no-encryption", 1);
    }
    
    await db.open();
    let query = `
    CREATE TABLE IF NOT EXISTS test (
      id INTEGER PRIMARY KEY NOT NULL,
      name TEXT NOT NULL
    );
    `

    const res: any = await db.execute(query);
    console.log(`res: ${JSON.stringify(res)}`);
    await db.close();
    await sqlite.closeConnection("db_issue9");
    
    ReactDOM.render(
      <React.StrictMode>
        <App /> 
      </React.StrictMode>,
      document.getElementById('root')
    );

    // If you want your app to work offline and load faster, you can change
    // unregister() to register() below. Note this comes with some pitfalls.
    // Learn more about service workers: https://bit.ly/CRA-PWA
    serviceWorker.unregister();

  } catch (err) {
    console.log(`Error: ${err}`);
    throw new Error(`Error: ${err}`)
  }

});
```

Then the `react-sqlite-hook` is initialized in the `App.tsx` file

```ts
import React, { useState, useRef }  from 'react';
import { Redirect, Route } from 'react-router-dom';
import {
  IonApp,
  IonIcon,
  IonLabel,
  IonRouterOutlet,
  IonTabBar,
  IonTabButton,
  IonTabs
} from '@ionic/react';
import { IonReactRouter } from '@ionic/react-router';
import { ellipse, square, triangle } from 'ionicons/icons';
import Tab1 from './pages/Tab1';
import Tab2 from './pages/Tab2';
import Tab3 from './pages/Tab3';
import { SQLiteHook, useSQLite } from 'react-sqlite-hook';
import ModalJsonMessages from './components/ModalJsonMessages';

/* Core CSS required for Ionic components to work properly */
import '@ionic/react/css/core.css';

/* Basic CSS for apps built with Ionic */
import '@ionic/react/css/normalize.css';
import '@ionic/react/css/structure.css';
import '@ionic/react/css/typography.css';

/* Optional CSS utils that can be commented out */
import '@ionic/react/css/padding.css';
import '@ionic/react/css/float-elements.css';
import '@ionic/react/css/text-alignment.css';
import '@ionic/react/css/text-transformation.css';
import '@ionic/react/css/flex-utils.css';
import '@ionic/react/css/display.css';

/* Theme variables */
import './theme/variables.css';


interface JsonListenerInterface {
  jsonListeners: boolean,
  setJsonListeners: React.Dispatch<React.SetStateAction<boolean>>,
}
interface existingConnInterface {
  existConn: boolean,
  setExistConn: React.Dispatch<React.SetStateAction<boolean>>,
}

// Singleton SQLite Hook
export let sqlite: SQLiteHook;
// Existing Connections Store
export let existingConn: existingConnInterface;
// Is Json Listeners used
export let isJsonListeners: JsonListenerInterface;

const App: React.FC = () => {
  const [existConn, setExistConn] = useState(false);
  existingConn = {existConn: existConn, setExistConn: setExistConn};
  const [jsonListeners, setJsonListeners] = useState(false);
  isJsonListeners = {jsonListeners: jsonListeners, setJsonListeners: setJsonListeners};
  const [isModal,setIsModal] = useState(false);
  const message = useRef("");
  
  const onProgressImport = async (progress: string) => {
    if(isJsonListeners.jsonListeners) {
      if(!isModal) setIsModal(true);
      message.current = message.current.concat(`${progress}\n`);
    }
  }
  const onProgressExport = async (progress: string) => {
    if(isJsonListeners.jsonListeners) {
      if(!isModal) setIsModal(true);
      message.current = message.current.concat(`${progress}\n`);
    }
  }
  
  // !!!!! if you do not want to use the progress events !!!!!
  // since react-sqlite-hook 2.1.0
  // sqlite = useSQLite()
  // before
  // sqlite = useSQLite({})
  // !!!!!                                               !!!!!

  sqlite = useSQLite({
    onProgressImport,
    onProgressExport
  });
  const handleClose = () => {
    setIsModal(false);
    message.current = "";
  }

  return (
  <IonApp>
    <IonReactRouter>
        <IonTabs>
          <IonRouterOutlet>
            <Route path="/tab1" component={Tab1} exact={true} />
            <Route path="/tab2" component={Tab2} exact={true} />
            <Route path="/tab3" component={Tab3} />
            <Route path="/" render={() => <Redirect to="/tab1" />} exact={true} />
          </IonRouterOutlet>
          <IonTabBar slot="bottom">
            <IonTabButton tab="tab1" href="/tab1">
              <IonIcon icon={triangle} />
              <IonLabel>Tab 1</IonLabel>
            </IonTabButton>
            <IonTabButton tab="tab2" href="/tab2">
              <IonIcon icon={ellipse} />
              <IonLabel>Tab 2</IonLabel>
            </IonTabButton>
            <IonTabButton tab="tab3" href="/tab3">
              <IonIcon icon={square} />
              <IonLabel>Tab 3</IonLabel>
            </IonTabButton>
          </IonTabBar>
        </IonTabs>
    </IonReactRouter>
    { isModal
      ? <ModalJsonMessages close={handleClose} message={message.current}></ModalJsonMessages>
      : null
    }
  </IonApp>
  )
};

export default App;
```

### React SQLite Hook Use in Components

- in a `component` file

```ts
import React, { useState, useEffect, useRef } from 'react';
import './Test2dbs.css';
import { IonCard,IonCardContent } from '@ionic/react';

import { createTablesNoEncryption, importTwoUsers } from '../Utils/noEncryptionUtils';
import { createSchemaContacts, setContacts } from '../Utils/encryptedSetUtils';
import { deleteDatabase } from '../Utils/deleteDBUtil';     
      
import { sqlite, existingConn } from '../App';
import { Dialog } from '@capacitor/dialog';

const Test2dbs: React.FC = () => {
  const [log, setLog] = useState<string[]>([]);
  const errMess = useRef("");
  const showAlert = async (message: string) => {
      await Dialog.alert({
        title: 'Error Dialog',
        message: message,
      });
  };

  useEffect( () => {
    const testtwodbs = async (): Promise<Boolean>  => {
      setLog((log) => log.concat("* Starting testTwoDBS *\n"));
      try {

        // initialize the connection
        const db = await sqlite
            .createConnection("testNew", false, "no-encryption", 1);
        const db1 = await sqlite
            .createConnection("testSet", true, "secret", 1);

        // check if the databases exist 
        // and delete it for multiple successive tests
        await deleteDatabase(db);
        await deleteDatabase(db1);

        // open db testNew
        await db.open();

        // create tables in db
        let ret: any = await db.execute(createTablesNoEncryption);
        if (ret.changes.changes < 0) {
            errMess.current = `Execute changes < 0`;
            return false;
        }

        // create synchronization table 
        ret = await db.createSyncTable();
        if (ret.changes.changes < 0) {
            errMess.current = `CreateSyncTable changes < 0`;
            return false;
        }

        // set the synchronization date
        const syncDate: string = "2020-11-25T08:30:25.000Z";
        await db.setSyncDate(syncDate);

        // add two users in db
        ret = await db.execute(importTwoUsers);
        if (ret.changes.changes !== 2) {
            errMess.current = `Execute importTwoUsers changes != 2`;
            return false;
        }
        // select all users in db
        ret = await db.query("SELECT * FROM users;");
        if(ret.values.length !== 2 || ret.values[0].name !== "Whiteley" ||
                            ret.values[1].name !== "Jones") {
            errMess.current = `Query not returning 2 values`;
            return false;
        }

        // open db testSet
        await db1.open();
        // create tables in db1
        ret = await db1.execute(createSchemaContacts);
        if (ret.changes.changes < 0) {
            errMess.current = `Execute createSchemaContacts changes < 0`;
            return false;
        }
        // load setContacts in db1
        ret = await db1.executeSet(setContacts);
        if (ret.changes.changes !== 5) {
            errMess.current = `ExecuteSet setContacts changes != 5`;
            return false;
        }

        // select users where company is NULL in db
        ret = await db.query("SELECT * FROM users WHERE company IS NULL;");
        if(ret.values.length !== 2 || ret.values[0].name !== "Whiteley" ||
                            ret.values[1].name !== "Jones") {
            errMess.current = `Query Company is NULL not returning 2 values`;
            return false;
        }
        // add one user with statement and values              
        let sqlcmd: string = 
            "INSERT INTO users (name,email,age) VALUES (?,?,?)";
        let values: Array<any>  = ["Simpson","Simpson@example.com",69];
        ret = await db.run(sqlcmd,values);
        if(ret.changes.lastId !== 3) {
            errMess.current = `Run lastId != 3`;
            return false;
        }
        // add one user with statement              
        sqlcmd = `INSERT INTO users (name,email,age) VALUES ` + 
                        `("Brown","Brown@example.com",15)`;
        ret = await db.run(sqlcmd);
        if(ret.changes.lastId !== 4) {
            errMess.current = `Run lastId != 4`;
            return false;
        }
        setLog((log) => log.concat("* Ending testTwoDBS *\n"));
        existingConn.setExistConn(true);
        return true;
      } catch (err) {
        errMess.current = `${err.message}`;
        return false;
      }
    }
    if(sqlite.isAvailable) {
      testtwodbs().then(async res => {
        if(res) {    
          setLog((log) => log
              .concat("\n* The set of tests was successful *\n"));
        } else {
          setLog((log) => log
              .concat("\n* The set of tests failed *\n"));
          await showAlert(errMess.current);
        }
      });
    } else {
      sqlite.getPlatform().then((ret: { platform: string; })  =>  {
          setLog((log) => log.concat("\n* Not available for " + 
                              ret.platform + " platform *\n"));
      });         
    }
  }, [errMess]);   

  return (
      <IonCard className="container-test2dbs">
          <IonCardContent>
              <pre>
                  <p>{log}</p>
              </pre>
              {errMess.current.length > 0 && <p>{errMess.current}</p>}
          </IonCardContent>
      </IonCard>
  );
};

export default Test2dbs;
```

Where

- `../Utils/noEncryptionUtils`

```ts
import { capSQLiteSet } from '@asello/capacitor-sqlite';
export const createTablesNoEncryption: string = `
    CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY NOT NULL,
    email TEXT UNIQUE NOT NULL,
    name TEXT,
    company TEXT,
    size REAL,
    age INTEGER,
    last_modified INTEGER DEFAULT (strftime('%s', 'now'))
    );
    CREATE TABLE IF NOT EXISTS messages (
    id INTEGER PRIMARY KEY NOT NULL,
    userid INTEGER,
    title TEXT NOT NULL,
    body TEXT NOT NULL,
    last_modified INTEGER DEFAULT (strftime('%s', 'now')),
    FOREIGN KEY (userid) REFERENCES users(id) ON DELETE SET DEFAULT
    );
    CREATE INDEX IF NOT EXISTS users_index_name ON users (name);
    CREATE INDEX IF NOT EXISTS users_index_last_modified ON users (last_modified);
    CREATE INDEX IF NOT EXISTS messages_index_last_modified ON messages (last_modified);
    CREATE TRIGGER IF NOT EXISTS users_trigger_last_modified 
    AFTER UPDATE ON users
    FOR EACH ROW WHEN NEW.last_modified <= OLD.last_modified  
    BEGIN  
        UPDATE users SET last_modified= (strftime('%s', 'now')) WHERE id=OLD.id;   
    END;      
    CREATE TRIGGER IF NOT EXISTS messages_trigger_last_modified AFTER UPDATE ON messages
    FOR EACH ROW WHEN NEW.last_modified <= OLD.last_modified  
    BEGIN  
        UPDATE messages SET last_modified= (strftime('%s', 'now')) WHERE id=OLD.id;   
    END;      
    PRAGMA user_version = 1;
`;
export const importTwoUsers: string = `
    DELETE FROM users;
    INSERT INTO users (name,email,age) VALUES ("Whiteley","Whiteley.com",30);
    INSERT INTO users (name,email,age) VALUES ("Jones","Jones.com",44);
`;
```

- `../Utils/encryptedSetUtils`

```ts
import { capSQLiteSet } from '@asello/capacitor-sqlite';
export const createSchemaContacts: string = `
CREATE TABLE IF NOT EXISTS contacts (
  id INTEGER PRIMARY KEY NOT NULL,
  email TEXT UNIQUE NOT NULL,
  name TEXT,
  FirstName TEXT,
  company TEXT,
  size REAL,
  age INTEGER,
  MobileNumber TEXT
);
CREATE INDEX IF NOT EXISTS contacts_index_name ON contacts (name);
CREATE INDEX IF NOT EXISTS contacts_index_email ON contacts (email);
PRAGMA user_version = 1;
`;
export const setContacts: Array<capSQLiteSet> = [
  {
    statement:
      'INSERT INTO contacts (name,FirstName,email,age,MobileNumber) VALUES (?,?,?,?,?);',
    values: ['Simpson', 'Tom', 'Simpson@example.com', 69, '4405060708'],
  },
  {
    statement:
      'INSERT INTO contacts (name,FirstName,email,age,MobileNumber) VALUES (?,?,?,?,?);',
    values: ['Jones', 'David', 'Jones@example.com', 42, '4404030201'],
  },
  {
    statement:
      'INSERT INTO contacts (name,FirstName,email,age,MobileNumber) VALUES (?,?,?,?,?);',
    values: ['Whiteley', 'Dave', 'Whiteley@example.com', 45, '4405162732'],
  },
  {
    statement:
      'INSERT INTO contacts (name,FirstName,email,age,MobileNumber) VALUES (?,?,?,?,?);',
    values: ['Brown', 'John', 'Brown@example.com', 35, '4405243853'],
  },
  {
    statement: 'UPDATE contacts SET age = ? , MobileNumber = ? WHERE id = ?;',
    values: [51, '4404030202', 2],
  },
];
```

- `../Utils/deleteDBUtil`

```ts
import { SQLiteDBConnection } from '@asello/capacitor-sqlite';

export async function deleteDatabase(db: SQLiteDBConnection): Promise<void> {
    try {
      let ret: any = await db.isExists();
      if(ret.result) {
        const dbName = db.getConnectionDBName();
        console.log("$$$ database " + dbName + " before delete");
        await db.delete();
        console.log("$$$ database " + dbName + " after delete " + ret.result);
        return Promise.resolve();
      } else {
        return Promise.resolve();
      }
    } catch (err) {
      return Promise.reject(err);
    }
}
```

- `ModalJsonMessages.tsx`

```ts
import React, { useState } from 'react';
import './ModalJsonMessages.css';
import Modal, { Styles } from 'react-modal';

interface ModalProps {
    message: string;
    close: any;
}
  
const ModalJsonMessages: React.FC<ModalProps> = (props) => {
    const [modalIsOpen,setModalIsOpen] = useState(true);

    const setModalIsOpenToFalse = () => {
        setModalIsOpen(false);
        props.close();
    }
    const customStyles: Styles = {
        content : {
          top                   : '10%',
          left                  : '2%',
          right                 : '2%',
          bottom                : '10%',
          backgroundColor       : '#D3D3D3',
          borderRadius          : '25px' ,
          whiteSpace            : 'pre-wrap',
          overflowWrap          : 'break-word' ,
          wordWrap              : 'break-word',
          hyphens               : 'auto',
        }
    };
    return (
        <Modal isOpen={modalIsOpen} style={customStyles} ariaHideApp={false}>
            <button className="button" onClick={setModalIsOpenToFalse}>Close</button>
            <pre>
                <p className="message">{props.message}</p>
            </pre>
        </Modal>
    );

};
export default ModalJsonMessages;

```
 - `ModalJsonMessages.css`

 ```
 .button {
    font-size: 16px;
    font-weight: bold;
}
.message {
    font-size: 14px;
}
 ```
