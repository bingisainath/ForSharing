import Realm from 'realm';
import { API_NAMES, BINARY_STRING } from '../Constant/Constants';
import { base64ToArrayBuffer } from '../Helper/Helper';
import LogManager from '../Helper/LogManager';
import { FavoriteGroupModel } from '../Model/FavouriteGroupModel';
import { DriveItemSchema, FavoriteGroupSchema, FavoriteSchema, LanguageDataSchema, LastModifyDateSchema, UserSchema } from './Schema';

export class DatabaseManager {
    //Constants
    kParentReferenceId = 'parentReferenceId';
    kUniqueId = 'uniqueId';
    kName = 'name';
    kFavoriteGroupName = 'favoriteGroupName';
    kWebUrl = 'webUrl';
    kMasterFolderName = 'Master';
    kRegionalFolderName = 'Regional';
    kDefaultFavoriteGroupName = 'Default';
    kCountryRoot = (country: string) => {
        return `webUrl == '${API_NAMES.ROOT_WEB_URL + country}'`;
    };

    realm: Realm;
    static dbInstance: DatabaseManager;

    constructor() {
        console.log('Database constructor');
        this.initializeRealm();
    }
    static getInstance() {
        if (this.dbInstance == null) {
            this.dbInstance = new DatabaseManager();
        }
        return this.dbInstance;
    }

    /**
     * Initialize Database with all schema
     */
    initializeRealm() {
        console.log('initializeRealm started');
        try {
            //https://www.convertstring.com/EncodeDecode/HexEncode pass encryption key to URL to get hexcode
            //use it to open Realm

            this.realm = new Realm({
                encryptionKey: base64ToArrayBuffer(BINARY_STRING),
                schema: [DriveItemSchema, UserSchema, FavoriteSchema, FavoriteGroupSchema,LastModifyDateSchema,LanguageDataSchema],
                schemaVersion: 1,
                migration: (_oldRealm, _newRealm) => {
                    //app already release to app store and in next version if you update any field name or change
                    //its data type then new app to work without failing we have to right code here in this
                },
            });

            console.log('initializeRealm finished');
        } catch (error) {
            console.log('initializeRealm error=', error);
        }
    }

    public createEntity = async (schemaName: string, data: any) => {
        try {
            this.realm.write(async () => {
                if (data?.length > 0) {
                    for (let i = 0, total = data.length; i < total; i++) {
                        this.realm.create(schemaName, data[i], Realm.UpdateMode.All);
                    }
                } else {
                    this.realm.create(schemaName, data, Realm.UpdateMode.All);
                }
            });
        } catch (error) {
            LogManager.info('createEntity--->', error);
            return '';
        }
    };

    public getEntities = (schemaName: string, query?: string) => {
        try {
            if (query) {
                let queryResult = this.realm.objects(schemaName).filtered(query);
                let copyOfJsonArray = Array.from(queryResult);
                return this.copyRealmObject(copyOfJsonArray);
            } else {
                let copyOfJsonArray = Array.from(this.realm.objects(schemaName));
                return this.copyRealmObject(copyOfJsonArray);
            }
        } catch (error) {
            LogManager.error('getEntities--->', error);
            return [];
        }
    };

    public getEntity = (schemaName: string, primaryKey: string) => {
        try {
            var group = this.realm?.objectForPrimaryKey(schemaName, primaryKey);
            return this.copyRealmObject(group);
        } catch (error) {
            LogManager.error('getEnttity--->', error);
            return null;
        }
    };

    public copyRealmObject = (item: any) => {
        return JSON.parse(JSON.stringify(item)); //refReplacer(item)};
    };

    public removeRealmObject = (schemaName: string, item: any) => {
        // delete entity
        try {
            this.realm?.write(() => {
                var group = this.realm?.objectForPrimaryKey(schemaName, item.id);
                if (group) {
                    this.realm?.delete(group);
                    group = undefined;
                }
            });
        } catch (error) {
            console.log(error);
        }
    };

    public deleteRealmObject = (schemaName: string, primaryKey: any) => {
        // delete entity
        try {
            this.realm?.write(() => {
                var group = this.realm?.objectForPrimaryKey(schemaName, primaryKey);
                if (group) {
                    this.realm?.delete(group);
                    group = undefined;
                }
            });
        } catch (error) {
            console.log(error);
        }
    };

    public deleteEntity = async (schema: SchemaType) => {
        try {
            this.realm.write(() => {
                this.realm.delete(this.realm.objects(schema));
            });
        } catch (error) {
            console.log('deleteEntity--->', error);
            return '';
        }
    };

}
