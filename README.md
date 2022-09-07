<p align="center"><br><img src="https://user-images.githubusercontent.com/236501/85893648-1c92e880-b7a8-11ea-926d-95355b8175c7.png" width="128" height="128" /></p>
<h3 align="center">SQLITE DATABASE</h3>
<p align="center"><strong><code>@asello/capacitor-sqlite</code></strong></p>
<br>
<p align="center" style="font-size:50px;color:red"><strong>CAPACITOR 3</strong></p><br>

<p align="center">
  Capacitor community plugin for Native and Electron SQLite Databases. In Native databases could be encrypted with SQLCipher
</p>
<br>
<p align="center">
  <img src="https://img.shields.io/maintenance/yes/2022?style=flat-square" />
  <a href="https://github.com/capacitor-community/sqlite/actions?query=workflow%3A%22CI%22"><img src="https://img.shields.io/github/workflow/status/capacitor-community/sqlite/CI?style=flat-square" /></a>
  <a href="https://www.npmjs.com/package/@asello/capacitor-sqlite"><img src="https://img.shields.io/npm/l/@asello/capacitor-sqlite?style=flat-square" /></a>
<br>
  <a href="https://www.npmjs.com/package/@asello/capacitor-sqlite"><img src="https://img.shields.io/npm/dw/@asello/capacitor-sqlite?style=flat-square" /></a>
  <a href="https://www.npmjs.com/package/@asello/capacitor-sqlite"><img src="https://img.shields.io/npm/v/@asello/capacitor-sqlite?style=flat-square" /></a>
<!-- ALL-CONTRIBUTORS-BADGE:START - Do not remove or modify this section -->
<a href="#contributors-"><img src="https://img.shields.io/badge/all%20contributors-16-orange?style=flat-square" /></a>
<!-- ALL-CONTRIBUTORS-BADGE:END -->
</p>

## Maintainers

| Maintainer        | GitHub                                    | Social |
| ----------------- | ----------------------------------------- | ------ |
| Quéau Jean Pierre | [jepiqueau](https://github.com/jepiqueau) |        |

## CAPACITOR 3 (Master)

🚨 Release 3.4.3-3 all platforms ->> 🚨

The main change is related to the delete table's rows when a synchronization table exists as well as a last_mofidied table's column, allowing for database synchronization of the local database with a remote server database.

- All existing triggers to YOUR_TABLE_NAME_trigger_last_modified must be modified as follows
  ```
  CREATE TRIGGER YOUR_TABLE_NAME_trigger_last_modified
    AFTER UPDATE ON YOUR_TABLE_NAME
    FOR EACH ROW WHEN NEW.last_modified < OLD.last_modified
    BEGIN
        UPDATE YOUR_TABLE_NAME SET last_modified= (strftime('%s', 'now')) WHERE id=OLD.id;
    END;
  ```
- an new column `sql_deleted` must be added to each of your tables as
  ```
  sql_deleted BOOLEAN DEFAULT 0 CHECK (sql_deleted IN (0, 1))
  ```
  This column will be autommatically set to 1 when you will use a `DELETE FROM ...` sql statement in the `execute`, `run` or `executeSet` methods.

- In the JSON object that you provide to `importFromJson`, all the deleted rows in your remote server database's tables must have the `sql_deleted` column set to 1. This will indicate to the import process to physically delete the corresponding rows in your local database. All the others rows must have the `sql_deleted` column set to 0. 

- In the JSON object outputs by the `exportToJson`, all the deleted rows in your local database have got the `sql_deleted` column set to 1 to help in your synchronization management process with the remote server database. A system `last_exported_date` is automatically saved in the synchronization table at the start of the export process flow.

- On successfull completion of your synchronization management process with the remote server database, you must 
  - Set a new synchronization date (as `(new Date()).toISOString()`) with the `setSyncDate` method.
  - Execute the `deleteExportedRows` method which physically deletes all table's rows having 1 as value for the `sql_deleted` column prior to the `last_exported_date` in your local database.

An example of using this new feature is given in [solidjs-vite-sqlite-app](https://github.com/jepiqueau/capacitor-solid-sqlite). It has been used to test the validity of the implementation.


🚨 Release 3.4.3-3 <<- 🚨

🚨 Release 3.4.2-4 ->> 🚨
!!!! DO NOT USE IT !!!!
🚨 Release 3.4.2-4 <<- 🚨

🚨 Since release 3.4.2-3 ->> 🚨

 - **overwrite** boolean parameter has been added to the Json Object (default false) 
   - `true` : delete the physically the database whatever the version is.
   - `false`: 
      - re-importing a database with the same `version` number will do nothing, keeping the existing database and will return `changes = 0`
      - re-importing a database with a lower `version` number will throw an error `ImportFromJson: Cannot import a version lower than `

 - During an import in `full` mode the `Foreign Key` constraint has been turn off before dropping the tables and turn back on after

🚨 Since release 3.4.2-3 <<- 🚨
🚨 Since release 3.4.1 ->> 🚨
  Databases location for Electron can be set in `the config.config.ts` as followed:

  - for sharing databases between users:

    ``` 
    plugins: {
      CapacitorSQLite: {
        electronMacLocation: "/YOUR_DATABASES_PATH",
        electronWindowsLocation: "C:\\ProgramData\\CapacitorDatabases",
        electronLinuxLocation: "/home/CapacitorDatabases"
      }
    }
    ``` 

  - for only the user in its Home folder

    ``` 
    Plugins: {
      CapacitorSQLite: {
        electronMacLocation: "Databases",
        electronWindowsLocation: "Databases",
        electronLinuxLocation: "Databases"
      }
    }
    ``` 

  For existing databases, YOU MUST COPY old databases to the new location
  You MUST remove the Electron folder and add it again with

  ``` 
  npx cap add @capacitor-community/electron
  npm run build 
  cd electron
  npm i --save sqlite3
  npm i --save @types:sqlite3
  npm run rebuild
  cd ..
  npx cap sync @capacitor-community/electron
  npm run build
  npx cap copy @capacitor-community/electron
  npx cap open @capacitor-community/electron
  ``` 
🚨 Since release 3.4.1 <<- 🚨

🚨 Since release 3.4.1-1 ->> 🚨

  - add iosIsEncryption, androidIsEncryption in capacitor.config.ts
    When your application use only `non-encrypted databases` set those parameter to false then iOS KeyChain & Android MasterKey are not defined.
    
🚨 Since release 3.4.1-1 <<- 🚨

🚨 Since release 3.4.0-2 ->> 🚨 

- iOS & Android only
  Adding biometric FaceID/TouchID to secure the pass phrase in the Keychain/SharedPreferences stores. see:
   [Biometric_Authentication](https://github.com/capacitor-community/sqlite/blob/master/docs/Biometric-Authentication.md)

- iOS only
  Fix identical pass phrase stored in the Keychain for differents applications using the plugin by adding an application prefix to the Keychain account.
  Before the account `"CapacitorSQLitePlugin"` was used and was the same for all applications.
  Now by adding `iosKeychainPrefix: 'YOUR_APP_NAME'`in the `capacitor.config.ts` of your application,
  the account will be `"YOUR_APP_NAME_CapacitorSQLitePlugin"`
  If you were having a pass phrase stored, first modify the `capacitor.config.ts` and then run the command `isSecretStored` which will manage the upgrade of the Keychain account. 
🚨 Since release 3.4.0-2 <<- 🚨

🚨 Since release 3.3.3-2 ->> 🚨

  - iOS only
    Support for a database location not visible to iTunes and backed up to iCloud.
    For this you must add to the `const config: CapacitorConfig` of the `capacitor.config.ts` file of your application the following:
    ```ts
      plugins: {
        CapacitorSQLite: {
          "iosDatabaseLocation": "Library/CapacitorDatabase"
        }
      }
    ``` 
    Pre-existing databases from the `Documents` folder will be moved to the new folder `Library/CapacitorDatabase` and your application will work as before.
    If you do not modify the `capacitor.config.ts` file of your application the databases will still reside in the `Documents` folder

🚨 Since release 3.3.3-2 <<- 🚨

🚨 Since release 3.2.5-2 ->> 🚨

  - support zip file in copyFromAssets method
  - add optional `overwrite` parameter (true/false) default to true

🚨 Since release 3.2.5-2 <<- 🚨

🚨 Since release 3.2.3-1 ->> 🚨

The `initWebStore` and `saveToStore` methods have been added to the Web platform.
 - The `initWebStore` has been added to fix the issue#172 and since then is `MANDATORY`
  ```js
  ...
  if(platform === "web") {
    await customElements.whenDefined('jeep-sqlite');
    const jeepSqliteEl = document.querySelector('jeep-sqlite');
    if(jeepSqliteEl != null) {
      await sqliteConnection.initWebStore()
      ...
    }
  }
  ...
  ```
 - the `saveToStore` allows to perform intermediate save of the database in case the browser needs to delete the cache.
 
🚨 Since release 3.2.3-1 <<- 🚨

The test has been achieved on:

- a [Ionic/Angular app](https://github.com/jepiqueau/angular-sqlite-app-starter)

- a [Ionic/React app](https://github.com/jepiqueau/react-sqlite-app-starter)

- a [Ionic/Vue app](https://github.com/jepiqueau/vue-sqlite-app-starter)

- a [React Vite app](https://github.com/jepiqueau/react-vite-sqlite-app)

- a [Vue Vite app](https://github.com/jepiqueau/vuevite-app)

- a [Vue TypeORM app](https://github.com/jepiqueau/vue-typeorm-app)


## Browser Support

The plugin follows the guidelines from the `Capacitor Team`,

- [Capacitor Browser Support](https://capacitorjs.com/docs/v3/web#browser-support)

meaning that it will not work in IE11 without additional JavaScript transformations, e.g. with [Babel](https://babeljs.io/).

## Installation

```bash
npm install @asello/capacitor-sqlite
npm run build
npx cap add android
npx cap add ios
npx cap add @capacitor-community/electron
```

and do when you update 

```bash
npx cap sync
npx cap sync @capacitor-community/electron
```

### Web

#### For Angular framework

- copy manually the file `sql-wasm.wasm` from `node_modules/sql.js/dist/sql-wasm.wasm` to the `src/assets` folder of YOUR_APP 

#### For Vue & React frameworks

- copy manually the file `sql-wasm.wasm` from `node_modules/sql.js/dist/sql-wasm.wasm` to the `public/assets` folder of YOUR_APP 

### IOS

- on iOS, no further steps needed.

### Android

- On Android, no further steps needed.


### Electron

- On Electron, go to the Electron folder of YOUR_APPLICATION

```bash
cd electron
npm install --save sqlite3
npm install --save jszip
npm install --save-dev @types/sqlite3
npm run build
```

## Build & Run

```
npm run build
npx cap copy
npx cap copy web
npx cap copy @capacitor-community/electron
```

### Web
- Angular
```
ionic serve
```
- Vue
```
npm run serve
```
- React
```
npm run start
```

### IOS

```
npx cap open ios
```

### Android

```
npx cap open android
```

### Electron

```
npx cap open @capacitor-community/electron
```

## Readme previous releases

[previous releases](https://github.com/capacitor-community/sqlite/readme_previous_release.md)

## Issues

[issues](https://github.com/capacitor-community/sqlite/issues)


## Configuration

No configuration required for this plugin

## Supported methods

| Name                        | Android | iOS | Electron | Web |
| :-------------------------- | :------ | :-- | :------- | :-- |
| createConnection            | ✅      | ✅  | ✅        | ✅  |
| closeConnection             | ✅      | ✅  | ✅        | ✅  |
| isConnection                | ✅      | ✅  | ✅        | ✅  |
| open (non-encrypted DB)     | ✅      | ✅  | ✅        | ✅  |
| open (encrypted DB)         | ✅      | ✅  | ❌        | ❌  |
| close                       | ✅      | ✅  | ✅        | ✅  |
| getUrl                      | ✅      | ✅  | ❌        | ❌  |
| getVersion                  | ✅      | ✅  | ✅        | ✅  |
| execute                     | ✅      | ✅  | ✅        | ✅  |
| executeSet                  | ✅      | ✅  | ✅        | ✅  |
| run                         | ✅      | ✅  | ✅        | ✅  |
| query                       | ✅      | ✅  | ✅        | ✅  |
| deleteDatabase              | ✅      | ✅  | ✅        | ✅  |
| importFromJson              | ✅      | ✅  | ✅        | ✅  |
| exportToJson                | ✅      | ✅  | ✅        | ✅  |
| deleteExportedRows          | ✅      | ✅  | ✅        | ✅  | NEW in 3.4.3-2
| createSyncTable             | ✅      | ✅  | ✅        | ✅  |
| setSyncDate                 | ✅      | ✅  | ✅        | ✅  |
| getSyncDate                 | ✅      | ✅  | ✅        | ✅  |
| isJsonValid                 | ✅      | ✅  | ✅        | ✅  |
| isDBExists                  | ✅      | ✅  | ✅        | ✅  |
| addUpgradeStatement         | ✅      | ✅  | ✅        | ✅  |
| copyFromAssets              | ✅      | ✅  | ✅        | ✅  |
| isDBOpen                    | ✅      | ✅  | ✅        | ✅  |
| isDatabase                  | ✅      | ✅  | ✅        | ✅  |
| isTableExists               | ✅      | ✅  | ✅        | ✅  |
| getTableList                | ✅      | ✅  | ✅        | ✅  |
| getDatabaseList             | ✅      | ✅  | ✅        | ✅  |
| getMigratableDbList         | ✅      | ✅  | ❌        | ❌  |
| addSQLiteSuffix             | ✅      | ✅  | ❌        | ❌  |
| deleteOldDatabases          | ✅      | ✅  | ❌        | ❌  |
| checkConnectionsConsistency | ✅      | ✅  | ✅        | ✅  |
| isSecretStored              | ✅      | ✅  | ❌        | ❌  |
| setEncryptionSecret         | ✅      | ✅  | ❌        | ❌  |
| changeEncryptionSecret      | ✅      | ✅  | ❌        | ❌  |
| initWebStore                | ❌      | ❌  | ❌        | ✅  |
| saveToStore                 | ❌      | ❌  | ❌        | ✅  |
| getNCDatabasePath           | ✅      | ✅  | ❌        | ❌  |
| createNCConnection          | ✅      | ✅  | ❌        | ❌  |
| closeNCConnection           | ✅      | ✅  | ❌        | ❌  |
| isNCDatabase                | ✅      | ✅  | ❌        | ❌  |
| transaction                 | ✅      | ✅  | ✅        | ✅  |

## Supported SQLite Types

 -[Datatypes In SQLite Version 3](https://www.sqlite.org/datatype3.html)

## Documentation

### API

- [API_Documentation](https://github.com/capacitor-community/sqlite/blob/master/docs/API.md)

- [API_Connection_Wrapper_Documentation](https://github.com/capacitor-community/sqlite/blob/master/docs/APIConnection.md)

- [API_DB_Connection_Wrapper_Documentation](https://github.com/capacitor-community/sqlite/blob/master/docs/APIDBConnection.md)

- [ImportExportJson_Documentation](https://github.com/capacitor-community/sqlite/blob/master/docs/ImportExportJson.md)

- [UpgradeDatabaseVersion_Documentation](https://github.com/capacitor-community/sqlite/blob/master/docs/UpgradeDatabaseVersion.md)

- [MigratingCordovaDatabases_Documentation](https://github.com/capacitor-community/sqlite/blob/master/docs/MigratingCordovaDatabases.md)

- [TypeORM_Documentation](https://github.com/capacitor-community/sqlite/blob/master/docs/TypeORM-Usage.md)

- [Web_Documentation](https://github.com/capacitor-community/sqlite/blob/master/docs/Web-Usage.md)

- [Non_Conformed_Databases_Documentation](https://github.com/capacitor-community/sqlite/blob/master/docs/NonConformedDatabases.md)

- [Biometric_Authentication](https://github.com/capacitor-community/sqlite/blob/master/docs/Biometric-Authentication.md)

### Framework's Usage 

- [Ionic/Angular_Usage_Documentation](https://github.com/capacitor-community/sqlite/blob/master/docs/Ionic-Angular-Usage.md)

- [Ionic/React_Usage_Documentation](https://github.com/capacitor-community/sqlite/blob/master/docs/Ionic-React-Usage.md)

- [Ionic/Vue_Usage_Documentation](https://github.com/capacitor-community/sqlite/blob/master/docs/Ionic-Vue-Usage.md)


## Applications demonstrating the use of the plugin

### Ionic/Angular

- [angular-sqlite-app-starter](https://github.com/jepiqueau/angular-sqlite-app-starter)

### Ionic/React

- [react-sqlite-app-starter](https://github.com/jepiqueau/react-sqlite-app-starter)

### React+Vite

- [react-vite-sqlite-app](https://github.com/jepiqueau/react-vite-sqlite-app)


### Ionic/Vue

- [vue-sqlite-app-starter](https://github.com/jepiqueau/vue-sqlite-app-starter)

### Vue

- [vue-sqlite-app](https://github.com/jepiqueau/vue-sqlite-app)

### Vue+Vite

- [vue-vite-sqlite-app](https://github.com/jepiqueau/vuevite-app)

### SolidJS+Vite

- [solidjs-vite-sqlite-app](https://github.com/jepiqueau/capacitor-solid-sqlite)


## Usage

- [see capacitor documentation](https://capacitor.ionicframework.com/docs/getting-started/with-ionic)


## Dependencies

The iOS and Android codes are using `SQLCipher` allowing for database encryption.
The iOS codes is using `ZIPFoundation` for unzipping assets files
The Electron code is using `sqlite3`.
The Web code is using the Stencil component `jeep-sqlite` based on `sql.js`, `localforage`. and `jszip`  

## Contributors ✨

Thanks goes to these wonderful people ([emoji key](https://allcontributors.org/docs/en/emoji-key)):

<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- prettier-ignore-start -->
<!-- markdownlint-disable -->
<table>
  <tr>
    <td align="center"><a href="https://github.com/jepiqueau"><img src="https://avatars3.githubusercontent.com/u/16580653?v=4" width="100px;" alt=""/><br /><sub><b>Jean Pierre Quéau</b></sub></a><br /><a href="https://github.com/capacitor-community/sqlite/commits?author=jepiqueau" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/paulantoine2"><img src="https://avatars0.githubusercontent.com/u/22882943?s=64&v=4" width="100px;" alt=""/><br /><sub><b>Paul Antoine</b></sub></a><br /><a href="https://github.com/capacitor-community/sqlite/commits?author=paulantoine2" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/karyfars"><img src="https://avatars2.githubusercontent.com/u/303016?s=60&u=1ce232ae3c22eac7b0b4778e46fe079939c39b40&v=4" width="100px;" alt=""/><br /><sub><b>Karyfars</b></sub></a><br /><a href="https://github.com/capacitor-community/sqlite/commits?author=karyfars" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/chriswep"><img src="https://avatars2.githubusercontent.com/u/1055809?s=400&u=e555940f143da8be255743028d6838cb5c020b44&v=4" width="100px;" alt=""/><br /><sub><b>Chriswep</b></sub></a><br /><a href="https://github.com/capacitor-community/sqlite/commits?author=chriswep" title="Documentation">📖</a></td>    
    <td align="center"><a href="https://github.com/nirajhinge"><img src="https://avatars.githubusercontent.com/u/54309996?v=4" width="100px;" alt=""/><br /><sub><b>Nirajhinge</b></sub></a><br /><a href="https://github.com/capacitor-community/sqlite/commits?author=nirajhinge" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/digaus"><img src="https://avatars.githubusercontent.com/u/15358538?v=4" width="100px;" alt=""/><br /><sub><b>Dirk Gausmann</b></sub></a><br /><a href="https://github.com/capacitor-community/sqlite/commits?author=digaus" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/IT-MikeS"><img src="https://avatars.githubusercontent.com/u/20338451?v=4" width="100px;" alt=""/><br /><sub><b>Mike Summerfeldt</b></sub></a><br /><a href="https://github.com/capacitor-community/sqlite/commits?author=IT-MikeS" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/peakcool"><img src="https://avatars.githubusercontent.com/u/14804014?v=4" width="100px;" alt=""/><br /><sub><b>Peakcool</b></sub></a><br /><a href="https://github.com/capacitor-community/sqlite/commits?author=peakcool" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/gion-andri"><img src="https://avatars.githubusercontent.com/u/540998?v=4" width="100px;" alt=""/><br /><sub><b>Gion-Andri Cantieni</b></sub></a><br /><a href="https://github.com/capacitor-community/sqlite/commits?author=gion-andri" title="Documentation">📖</a></td>
    <td align="center"><a href="https://github.com/robingenz"><img src="https://avatars.githubusercontent.com/u/13857929?v=4" width="100px;" alt=""/><br /><sub><b>Robin Genz</b></sub></a><br /><a href="https://github.com/capacitor-community/sqlite/commits?author=robingenz" title="Documentation">📖</a></td>
    <td align="center"><a href="https://github.com/dewald-els"><img src="https://avatars.githubusercontent.com/u/10051247?v=4" width="100px;" alt=""/><br /><sub><b>Dewald Els</b></sub></a><br /><a href="https://github.com/capacitor-community/sqlite/commits?author=dewald-els" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/joewoodhouse"><img src="https://avatars.githubusercontent.com/u/3168135?v=4" width="100px;" alt=""/><br /><sub><b>Joe Woodhouse</b></sub></a><br /><a href="https://github.com/capacitor-community/sqlite/commits?author=joewoodhouse" title="Documentation">📖</a></td>
    <td align="center"><a href="https://github.com/ptasheq"><img src="https://avatars.githubusercontent.com/u/3025106?v=4" width="100px;" alt=""/><br /><sub><b>Ptasheq</b></sub></a><br /><a href="https://github.com/capacitor-community/sqlite/commits?author=ptasheq" title="Documentation">📖</a></td>  
    <td align="center"><a href="https://github.com/victorybiz"><img src="https://avatars.githubusercontent.com/u/8276466?v=4" width="100px;" alt=""/><br /><sub><b>Victory Osayi</b></sub></a><br /><a href="https://github.com/capacitor-community/sqlite/commits?author=victorybiz" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/tobiasmuecksch"><img src="https://avatars.githubusercontent.com/u/1294854?v=4" width="100px;" alt=""/><br /><sub><b>Tobias Mücksch</b></sub></a><br /><a href="https://github.com/capacitor-community/sqlite/commits?author=tobiasmuecksch" title="Code">💻</a></td>
    <td align="center"><a href="https://github.com/dragermrb"><img src="https://avatars.githubusercontent.com/u/11479696?v=4" width="100px;" alt=""/><br /><sub><b>Manuel Rodríguez</b></sub></a><br /><a href="https://github.com/capacitor-community/sqlite/commits?author=dragermrb" title="Code">💻</a></td>
  </tr>
</table>

<!-- markdownlint-enable -->
<!-- prettier-ignore-end -->

<!-- ALL-CONTRIBUTORS-LIST:END -->

This project follows the [all-contributors](https://github.com/all-contributors/all-contributors) specification. Contributions of any kind welcome!
