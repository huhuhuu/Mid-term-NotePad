# Mid-term-NotePad
The mid-term work.
## Description: 
NotePad.
## Core code:
```
sNotesProjectionMap.put(NotePad.Notes._ID, NotePad.Notes._ID);

        // Maps "title" to "title"
        sNotesProjectionMap.put(NotePad.Notes.COLUMN_NAME_TITLE, NotePad.Notes.COLUMN_NAME_TITLE);

        // Maps "note" to "note"
        sNotesProjectionMap.put(NotePad.Notes.COLUMN_NAME_NOTE, NotePad.Notes.COLUMN_NAME_NOTE);

        // Maps "created" to "created"
        sNotesProjectionMap.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE,
                NotePad.Notes.COLUMN_NAME_CREATE_DATE);

        // Maps "modified" to "modified"
        sNotesProjectionMap.put(
                NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
                NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE);

        sNotesProjectionMap.put(
                NotePad.Notes.COLUMN_NAME_BACK_COLOR,
                NotePad.Notes.COLUMN_NAME_BACK_COLOR);
        sNotesProjectionMap.put(
                NotePad.Notes.COLUMN_NAME_FONT_SIZE,
                NotePad.Notes.COLUMN_NAME_FONT_SIZE);
                
                public void onCreate(SQLiteDatabase db) {
           db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
                   + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
                   + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
                   + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER,"
                   + NotePad.Notes.COLUMN_NAME_BACK_COLOR + " INTEGER,"
                   + NotePad.Notes.COLUMN_NAME_FONT_SIZE + " INTEGER"
                   + ");");
       }
       
        String orderBy;
       // If no sort order is specified, uses the default
       if (TextUtils.isEmpty(sortOrder)) {
           orderBy = NotePad.Notes.DEFAULT_SORT_ORDER;
       } else {
           // otherwise, uses the incoming sort order
           orderBy = sortOrder;
       }

       // Opens the database object in "read" mode, since no writes need to be done.
       SQLiteDatabase db = mOpenHelper.getReadableDatabase();

       /*
        * Performs the query. If no problems occur trying to read the database, then a Cursor
        * object is returned; otherwise, the cursor variable contains null. If no records were
        * selected, then the Cursor object is empty, and Cursor.getCount() returns 0.
        */
       Cursor c = qb.query(
           db,            // The database to query
           projection,    // The columns to return from the query
           selection,     // The columns for the where clause
           selectionArgs, // The values for the where clause
           null,          // don't group the rows
           null,          // don't filter by row groups
           orderBy        // The sort order
       );

       // Tells the Cursor what URI to watch, so it knows when its source data changes
       c.setNotificationUri(getContext().getContentResolver(), uri);
       
 public Uri insert(Uri uri, ContentValues initialValues) {

        // Validates the incoming URI. Only the full provider URI is allowed for inserts.
        if (sUriMatcher.match(uri) != NOTES) {
            throw new IllegalArgumentException("Unknown URI " + uri);
        }

        // A map to hold the new record's values.
        ContentValues values;

        // If the incoming values map is not null, uses it for the new values.
        if (initialValues != null) {
            values = new ContentValues(initialValues);

        } else {
            // Otherwise, create a new value map
            values = new ContentValues();
        }

        // Gets the current system time in milliseconds
        SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        String now=format.format(new Date());

        // If the values map doesn't contain the creation date, sets the value to the current time.
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_CREATE_DATE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE, now);
        }

        // If the values map doesn't contain the modification date, sets the value to the current
        // time.
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, now);
        }

        // If the values map doesn't contain a title, sets the value to the default title.
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_TITLE) == false) {
            Resources r = Resources.getSystem();
            values.put(NotePad.Notes.COLUMN_NAME_TITLE, r.getString(android.R.string.untitled));
        }

        // If the values map doesn't contain note text, sets the value to an empty string.
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_NOTE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_NOTE, "");
        }

        // 新建笔记，背景默认为白色
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_BACK_COLOR) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, NotePad.Notes.DEFAULT_COLOR);
        }
        // 默认字体设置
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_FONT_SIZE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_FONT_SIZE, NotePad.Notes.FONT_30);
        }
        // Opens the database object in "write" mode.
        SQLiteDatabase db = mOpenHelper.getWritableDatabase();

        // Performs the insert and returns the ID of the new note.
        long rowId = db.insert(
            NotePad.Notes.TABLE_NAME,        // The table to insert into.
            NotePad.Notes.COLUMN_NAME_NOTE,  // A hack, SQLite sets this column value to null
                                             // if values is empty.
            values                           // A map of column names, and the values to insert
                                             // into the columns.
        );

        // If the insert succeeded, the row ID exists.
        if (rowId > 0) {
            // Creates a URI with the note ID pattern and the new row ID appended to it.
            Uri noteUri = ContentUris.withAppendedId(NotePad.Notes.CONTENT_ID_URI_BASE, rowId);

            // Notifies observers registered against this provider that the data changed.
            getContext().getContentResolver().notifyChange(noteUri, null);
            return noteUri;
        }

        // If the insert didn't succeed, then the rowID is <= 0. Throws an exception.
        throw new SQLException("Failed to insert row into " + uri);
    }
    
    public int update(Uri uri, ContentValues values, String where, String[] whereArgs) {

        String x = (String)values.get(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE);
        // Opens the database object in "write" mode.
        SQLiteDatabase db = mOpenHelper.getWritableDatabase();
        int count;
        String finalWhere;

        // Does the update based on the incoming URI pattern
        switch (sUriMatcher.match(uri)) {

            // If the incoming URI matches the general notes pattern, does the update based on
            // the incoming data.
            case NOTES:

                // Does the update and returns the number of rows updated.
                count = db.update(
                    NotePad.Notes.TABLE_NAME, // The database table name.
                    values,                   // A map of column names and new values to use.
                    where,                    // The where clause column names.
                    whereArgs                 // The where clause column values to select on.
                );
                break;

            // If the incoming URI matches a single note ID, does the update based on the incoming
            // data, but modifies the where clause to restrict it to the particular note ID.
            case NOTE_ID:
                // From the incoming URI, get the note ID
                String noteId = uri.getPathSegments().get(NotePad.Notes.NOTE_ID_PATH_POSITION);

                /*
                 * Starts creating the final WHERE clause by restricting it to the incoming
                 * note ID.
                 */
                finalWhere =
                        NotePad.Notes._ID +                              // The ID column name
                        " = " +                                          // test for equality
                        uri.getPathSegments().                           // the incoming note ID
                            get(NotePad.Notes.NOTE_ID_PATH_POSITION)
                ;

                // If there were additional selection criteria, append them to the final WHERE
                // clause
                if (where !=null) {
                    finalWhere = finalWhere + " AND " + where;
                }


                // Does the update and returns the number of rows updated.
                count = db.update(
                    NotePad.Notes.TABLE_NAME, // The database table name.
                    values,                   // A map of column names and new values to use.
                    finalWhere,               // The final WHERE clause to use
                                              // placeholders for whereArgs
                    whereArgs                 // The where clause column values to select on, or
                                              // null if the values are in the where argument.
                );
                break;
            // If the incoming pattern is invalid, throws an exception.
            default:
                throw new IllegalArgumentException("Unknown URI " + uri);
        }
        
protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search_list);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        SearchView searchview = (SearchView) findViewById(R.id.search_view);

        if (searchview != null) {
            try {        //--拿到字节码
                Class<?> argClass = searchview.getClass();
                //--指定某个私有属性,mSearchPlate是搜索框父布局的名字
                Field ownField = argClass.getDeclaredField("mSearchPlate");
                //--暴力反射,只有暴力反射才能拿到私有属性
                ownField.setAccessible(true);
                View mView = (View) ownField.get(searchview);
                //--设置背景
                mView.setBackgroundColor(Color.TRANSPARENT);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }

        //为查询文本框注册监听器
        searchview.setOnQueryTextListener(NoteSearch.this);
    }

    @Override
    public boolean onQueryTextSubmit(String query) {
        return false;
    }

    @Override
    public boolean onQueryTextChange(String newText) {
        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " Like ? ";
        String[] selectionArgs = {"%" + newText + "%"};
        Cursor cursor = managedQuery(
                getIntent().getData(),            // Use the default content URI for the provider.
                PROJECTION,                       // Return the note ID and title for each note. and modifcation date
                selection,                        // 条件左边
                selectionArgs,                    // 条件右边
                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
        );
        if(cursor.getCount()==0){
            Toast.makeText(this,"sorry,no such note",Toast.LENGTH_SHORT).show();

        }
        String[] dataColumns = {NotePad.Notes.COLUMN_NAME_TITLE, NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE};
        int[] viewIDs = {android.R.id.text1, R.id.Text_time};
        YanCursorAdapter adapter = new YanCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
        );
        setListAdapter(adapter);
        return true;
    }

    @Override
    protected void onListItemClick(ListView l, View v, int position, long id) {
        // Constructs a new URI from the incoming URI and the row ID
        Uri uri = ContentUris.withAppendedId(getIntent().getData(), id);
        // Gets the action from the incoming Intent
        String action = getIntent().getAction();
        // Handles requests for note data
        if (Intent.ACTION_PICK.equals(action) || Intent.ACTION_GET_CONTENT.equals(action)) {
            // Sets the result to return to the component that called this Activity. The
            // result contains the new URI
            setResult(RESULT_OK, new Intent().setData(uri));
        } else {
            // Sends out an Intent to start an Activity that can handle ACTION_EDIT. The
            // Intent's data is the note ID URI. The effect is to call NoteEdit.
            startActivity(new Intent(Intent.ACTION_EDIT, uri));
        }
    }
    
public class NotesList extends ListActivity {

    // For logging and debugging
    private static final String TAG = "NotesList";

    /**
     * The columns needed by the cursor adapter
     */
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID,
            NotePad.Notes.COLUMN_NAME_TITLE,
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
            NotePad.Notes.COLUMN_NAME_BACK_COLOR,
            NotePad.Notes.COLUMN_NAME_FONT_SIZE
    };

    /** The index of the title column */
    private static final int COLUMN_INDEX_TITLE = 1;

    /**
     * onCreate is called when Android starts this Activity from scratch.
     */
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // The user does not need to hold down the key to use menu shortcuts.
        setDefaultKeyMode(DEFAULT_KEYS_SHORTCUT);

        /* If no data is given in the Intent that started this Activity, then this Activity
         * was started when the intent filter matched a MAIN action. We should use the default
         * provider URI.
         */
        // Gets the intent that started this Activity.
        Intent intent = getIntent();

        // If there is no data associated with the Intent, sets the data to the default URI, which
        // accesses a list of notes.
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }

        /*
         * Sets the callback for context menu activation for the ListView. The listener is set
         * to be this Activity. The effect is that context menus are enabled for items in the
         * ListView, and the context menu is handled by a method in NotesList.
         */
        getListView().setOnCreateContextMenuListener(this);

        /* Performs a managed query. The Activity handles closing and requerying the cursor
         * when needed.
         *
         * Please see the introductory note about performing provider operations on the UI thread.
         */
        Cursor cursor = managedQuery(
                getIntent().getData(),            // Use the default content URI for the provider.
                PROJECTION,                       // Return the note ID and title for each note.
                null,                             // No where clause, return all records.
                null,                             // No where clause, therefore no where column values.
                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
        );
        /*
         * The following two arrays create a "map" between columns in the cursor and view IDs
         * for items in the ListView. Each element in the dataColumns array represents
         * a column name; each element in the viewID array represents the ID of a View.
         * The SimpleCursorAdapter maps them in ascending order to determine where each column
         * value will appear in the ListView.
         */

        // The names of the cursor columns to display in the view, initialized to the title column
        String[] dataColumns = {
                NotePad.Notes.COLUMN_NAME_TITLE,
                NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE
        } ;

        // The view IDs that will display the cursor columns, initialized to the TextView in
        // noteslist_item.xml
        int[] viewIDs = {
                android.R.id.text1,
                R.id.Text_time
        };

        // Creates the backing adapter for the ListView.
        YanCursorAdapter adapter = new YanCursorAdapter(
                this,                             // The Context for the ListView
                R.layout.noteslist_item,          // Points to the XML for a list item
                cursor,                           // The cursor to get items from
                dataColumns,
                viewIDs
        );

        // Sets the ListView's adapter to be the cursor adapter that was just created.
        setListAdapter(adapter);
    }

    /**
     * Called when the user clicks the device's Menu button the first time for
     * this Activity. Android passes in a Menu object that is populated with items.
     *
     * Sets up a menu that provides the Insert option plus a list of alternative actions for
     * this Activity. Other applications that want to handle notes can "register" themselves in
     * Android by providing an intent filter that includes the category ALTERNATIVE and the
     * mimeTYpe NotePad.Notes.CONTENT_TYPE. If they do this, the code in onCreateOptionsMenu()
     * will add the Activity that contains the intent filter to its list of options. In effect,
     * the menu will offer the user other applications that can handle notes.
     * @param menu A Menu object, to which menu items should be added.
     * @return True, always. The menu should be displayed.
     */
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate menu from XML resource
        MenuInflater inflater = getMenuInflater();
        inflater.inflate(R.menu.list_options_menu, menu);

        // Generate any additional actions that can be performed on the
        // overall list.  In a normal install, there are no additional
        // actions found here, but this allows other applications to extend
        // our menu with their own actions.
        Intent intent = new Intent(null, getIntent().getData());
        intent.addCategory(Intent.CATEGORY_ALTERNATIVE);
        menu.addIntentOptions(Menu.CATEGORY_ALTERNATIVE, 0, 0,
                new ComponentName(this, NotesList.class), null, intent, 0, null);

        return super.onCreateOptionsMenu(menu);
    }

    @Override
    public boolean onPrepareOptionsMenu(Menu menu) {
        super.onPrepareOptionsMenu(menu);

        // The paste menu item is enabled if there is data on the clipboard.
        ClipboardManager clipboard = (ClipboardManager)
                getSystemService(Context.CLIPBOARD_SERVICE);


        MenuItem mPasteItem = menu.findItem(R.id.menu_paste);

        // If the clipboard contains an item, enables the Paste option on the menu.
        if (clipboard.hasPrimaryClip()) {
            mPasteItem.setEnabled(true);
        } else {
            // If the clipboard is empty, disables the menu's Paste option.
            mPasteItem.setEnabled(false);
        }

        // Gets the number of notes currently being displayed.
        final boolean haveItems = getListAdapter().getCount() > 0;

        // If there are any notes in the list (which implies that one of
        // them is selected), then we need to generate the actions that
        // can be performed on the current selection.  This will be a combination
        // of our own specific actions along with any extensions that can be
        // found.
        if (haveItems) {

            // This is the selected item.
            Uri uri = ContentUris.withAppendedId(getIntent().getData(), getSelectedItemId());

            // Creates an array of Intents with one element. This will be used to send an Intent
            // based on the selected menu item.
            Intent[] specifics = new Intent[1];

            // Sets the Intent in the array to be an EDIT action on the URI of the selected note.
            specifics[0] = new Intent(Intent.ACTION_EDIT, uri);

            // Creates an array of menu items with one element. This will contain the EDIT option.
            MenuItem[] items = new MenuItem[1];

            // Creates an Intent with no specific action, using the URI of the selected note.
            Intent intent = new Intent(null, uri);

            /* Adds the category ALTERNATIVE to the Intent, with the note ID URI as its
             * data. This prepares the Intent as a place to group alternative options in the
             * menu.
             */
            intent.addCategory(Intent.CATEGORY_ALTERNATIVE);

            /*
             * Add alternatives to the menu
             */
            menu.addIntentOptions(
                    Menu.CATEGORY_ALTERNATIVE,  // Add the Intents as options in the alternatives group.
                    Menu.NONE,                  // A unique item ID is not required.
                    Menu.NONE,                  // The alternatives don't need to be in order.
                    null,                       // The caller's name is not excluded from the group.
                    specifics,                  // These specific options must appear first.
                    intent,                     // These Intent objects map to the options in specifics.
                    Menu.NONE,                  // No flags are required.
                    items                       // The menu items generated from the specifics-to-
                    // Intents mapping
            );
            // If the Edit menu item exists, adds shortcuts for it.
            if (items[0] != null) {

                // Sets the Edit menu item shortcut to numeric "1", letter "e"
                items[0].setShortcut('1', 'e');
            }
        } else {
            // If the list is empty, removes any existing alternative actions from the menu
            menu.removeGroup(Menu.CATEGORY_ALTERNATIVE);
        }

        // Displays the menu
        return true;
    }

    /**
     * This method is called when the user selects an option from the menu, but no item
     * in the list is selected. If the option was INSERT, then a new Intent is sent out with action
     * ACTION_INSERT. The data from the incoming Intent is put into the new Intent. In effect,
     * this triggers the NoteEditor activity in the NotePad application.
     *
     * If the item was not INSERT, then most likely it was an alternative option from another
     * application. The parent method is called to process the item.
     * @param item The menu item that was selected by the user
     * @return True, if the INSERT menu item was selected; otherwise, the result of calling
     * the parent method.
     */
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case R.id.menu_add:
                /*
                 * Launches a new Activity using an Intent. The intent filter for the Activity
                 * has to have action ACTION_INSERT. No category is set, so DEFAULT is assumed.
                 * In effect, this starts the NoteEditor Activity in NotePad.
                 */
                startActivity(new Intent(Intent.ACTION_INSERT, getIntent().getData()));
                return true;
            case R.id.menu_paste:
                /*
                 * Launches a new Activity using an Intent. The intent filter for the Activity
                 * has to have action ACTION_PASTE. No category is set, so DEFAULT is assumed.
                 * In effect, this starts the NoteEditor Activity in NotePad.
                 */
                startActivity(new Intent(Intent.ACTION_PASTE, getIntent().getData()));
                return true;
            case R.id.menu_search:
                Intent intent = new Intent();
                intent.setClass(NotesList.this,NoteSearch.class);
                NotesList.this.startActivity(intent);
                return true;
            default:
                return super.onOptionsItemSelected(item);
        }
    }

    /**
     * This method is called when the user context-clicks a note in the list. NotesList registers
     * itself as the handler for context menus in its ListView (this is done in onCreate()).
     *
     * The only available options are COPY and DELETE.
     *
     * Context-click is equivalent to long-press.
     *
     * @param menu A ContexMenu object to which items should be added.
     * @param view The View for which the context menu is being constructed.
     * @param menuInfo Data associated with view.
     * @throws ClassCastException
     */
    @Override
    public void onCreateContextMenu(ContextMenu menu, View view, ContextMenuInfo menuInfo) {

        // The data from the menu item.
        AdapterView.AdapterContextMenuInfo info;

        // Tries to get the position of the item in the ListView that was long-pressed.
        try {
            // Casts the incoming data object into the type for AdapterView objects.
            info = (AdapterView.AdapterContextMenuInfo) menuInfo;
        } catch (ClassCastException e) {
            // If the menu object can't be cast, logs an error.
            Log.e(TAG, "bad menuInfo", e);
            return;
        }

        /*
         * Gets the data associated with the item at the selected position. getItem() returns
         * whatever the backing adapter of the ListView has associated with the item. In NotesList,
         * the adapter associated all of the data for a note with its list item. As a result,
         * getItem() returns that data as a Cursor.
         */
        Cursor cursor = (Cursor) getListAdapter().getItem(info.position);

        // If the cursor is empty, then for some reason the adapter can't get the data from the
        // provider, so returns null to the caller.
        if (cursor == null) {
            // For some reason the requested item isn't available, do nothing
            return;
        }

        // Inflate menu from XML resource
        MenuInflater inflater = getMenuInflater();
        inflater.inflate(R.menu.list_context_menu, menu);

        // Sets the menu header to be the title of the selected note.
        menu.setHeaderTitle(cursor.getString(COLUMN_INDEX_TITLE));

        // Append to the
        // menu items for any other activities that can do stuff with it
        // as well.  This does a query on the system for any activities that
        // implement the ALTERNATIVE_ACTION for our data, adding a menu item
        // for each one that is found.
        Intent intent = new Intent(null, Uri.withAppendedPath(getIntent().getData(),
                Integer.toString((int) info.id) ));
        intent.addCategory(Intent.CATEGORY_ALTERNATIVE);
        menu.addIntentOptions(Menu.CATEGORY_ALTERNATIVE, 0, 0,
                new ComponentName(this, NotesList.class), null, intent, 0, null);
    }

    /**
     * This method is called when the user selects an item from the context menu
     * (see onCreateContextMenu()). The only menu items that are actually handled are DELETE and
     * COPY. Anything else is an alternative option, for which default handling should be done.
     *
     * @param item The selected menu item
     * @return True if the menu item was DELETE, and no default processing is need, otherwise false,
     * which triggers the default handling of the item.
     * @throws ClassCastException
     */
    @Override
    public boolean onContextItemSelected(MenuItem item) {
        // The data from the menu item.
        AdapterView.AdapterContextMenuInfo info;

        /*
         * Gets the extra info from the menu item. When an note in the Notes list is long-pressed, a
         * context menu appears. The menu items for the menu automatically get the data
         * associated with the note that was long-pressed. The data comes from the provider that
         * backs the list.
         *
         * The note's data is passed to the context menu creation routine in a ContextMenuInfo
         * object.
         *
         * When one of the context menu items is clicked, the same data is passed, along with the
         * note ID, to onContextItemSelected() via the item parameter.
         */
        try {
            // Casts the data object in the item into the type for AdapterView objects.
            info = (AdapterView.AdapterContextMenuInfo) item.getMenuInfo();
        } catch (ClassCastException e) {

            // If the object can't be cast, logs an error
            Log.e(TAG, "bad menuInfo", e);

            // Triggers default processing of the menu item.
            return false;
        }
        // Appends the selected note's ID to the URI sent with the incoming Intent.
        Uri noteUri = ContentUris.withAppendedId(getIntent().getData(), info.id);

        /*
         * Gets the menu item's ID and compares it to known actions.
         */
        switch (item.getItemId()) {
            case R.id.context_open:
                // Launch activity to view/edit the currently selected item
                startActivity(new Intent(Intent.ACTION_EDIT, noteUri));
                return true;
//BEGIN_INCLUDE(copy)
            case R.id.context_copy:
                // Gets a handle to the clipboard service.
                ClipboardManager clipboard = (ClipboardManager)
                        getSystemService(Context.CLIPBOARD_SERVICE);

                // Copies the notes URI to the clipboard. In effect, this copies the note itself
                clipboard.setPrimaryClip(ClipData.newUri(   // new clipboard item holding a URI
                        getContentResolver(),               // resolver to retrieve URI info
                        "Note",                             // label for the clip
                        noteUri)                            // the URI
                );

                // Returns to the caller and skips further processing.
                return true;
//END_INCLUDE(copy)
            case R.id.context_delete:

                // Deletes the note from the provider by passing in a URI in note ID format.
                // Please see the introductory note about performing provider operations on the
                // UI thread.
                getContentResolver().delete(
                        noteUri,  // The URI of the provider
                        null,     // No where clause is needed, since only a single note ID is being
                        // passed in.
                        null      // No where clause is used, so no where arguments are needed.
                );

                // Returns to the caller and skips further processing.
                return true;
            default:
                return super.onContextItemSelected(item);
        }
    }

    /**
     * This method is called when the user clicks a note in the displayed list.
     *
     * This method handles incoming actions of either PICK (get data from the provider) or
     * GET_CONTENT (get or create data). If the incoming action is EDIT, this method sends a
     * new Intent to start NoteEditor.
     * @param l The ListView that contains the clicked item
     * @param v The View of the individual item
     * @param position The position of v in the displayed list
     * @param id The row ID of the clicked item
     */
    @Override
    protected void onListItemClick(ListView l, View v, int position, long id) {

        // Constructs a new URI from the incoming URI and the row ID
        Uri uri = ContentUris.withAppendedId(getIntent().getData(), id);

        // Gets the action from the incoming Intent
        String action = getIntent().getAction();

        // Handles requests for note data
        if (Intent.ACTION_PICK.equals(action) || Intent.ACTION_GET_CONTENT.equals(action)) {

            // Sets the result to return to the component that called this Activity. The
            // result contains the new URI
            setResult(RESULT_OK, new Intent().setData(uri));
        } else {

            // Sends out an Intent to start an Activity that can handle ACTION_EDIT. The
            // Intent's data is the note ID URI. The effect is to call NoteEdit.
            startActivity(new Intent(Intent.ACTION_EDIT, uri));
        }
    }
package com.example.android.notepad;

import android.content.Context;
import android.database.Cursor;
import android.graphics.Color;
import android.view.View;
import android.widget.EditText;
import android.widget.SimpleCursorAdapter;
import android.widget.TextView;

public class YanCursorAdapter extends SimpleCursorAdapter {
    public YanCursorAdapter(Context context, int layout, Cursor c,
                            String[] from, int[] to) {
        super(context, layout, c, from, to);
    }
    @Override
    public void bindView(View view, Context context, Cursor cursor){
        super.bindView(view, context, cursor);

        //从数据库中读取的cursor中获取笔记列表对应的颜色数据，并设置笔记颜色
        int x = cursor.getInt(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
        /**
         * 银河白 255 255 255
         * 杏仁黄 250 249 222
         * 宁静蓝 137 171 227
         * 护眼绿 204 232 207
         * 蔷薇粉 242 121 222
         * 紫色 162 32 240
         * 灰色 192 192 192
         */
        switch (x){
            case NotePad.Notes.DEFAULT_COLOR:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
            case NotePad.Notes.YELLOW_COLOR:
                view.setBackgroundColor(Color.rgb(250, 249, 222));
                break;
            case NotePad.Notes.BLUE_COLOR:
                view.setBackgroundColor(Color.rgb(137, 171, 227));
                break;
            case NotePad.Notes.GREEN_COLOR:
                view.setBackgroundColor(Color.rgb(204, 232, 207));
                break;
            case NotePad.Notes.PURPLE_COLOR:
                view.setBackgroundColor(Color.rgb(162, 32, 240));
                break;
            case NotePad.Notes.GREY_COLOR:
                view.setBackgroundColor(Color.rgb(192, 192, 192));
                break;
            case NotePad.Notes.RED_COLOR:
                view.setBackgroundColor(Color.rgb(242, 221, 222));
                break;
            default:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
        }

    }
}

<LinearLayout  xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <TextView
    android:id="@android:id/text1"
    android:layout_width="match_parent"
    android:layout_height="?android:attr/listPreferredItemHeight"
    android:textAppearance="?android:attr/textAppearanceLarge"
    android:gravity="center_vertical"
    android:paddingLeft="5dip"
    android:singleLine="true" />
    <TextView
        android:id="@+id/text1_time"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="right"
        android:textAppearance="?android:attr/textAppearanceSmall"
        android:paddingLeft="5dip"
        android:textColor="@color/colorBlack"/>
</LinearLayout>

<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    >
    <item android:id="@+id/menu_save"
        android:icon="@drawable/ic_menu_save"
        android:alphabeticShortcut='s'
        android:title="@string/menu_save"
        android:showAsAction="always" />
    <item android:id="@+id/menu_revert"
        android:icon="@drawable/ic_menu_revert"
        android:title="@string/menu_revert" />
    <item android:id="@+id/menu_color"
        android:icon="@drawable/color"
        android:title="Color"
        android:showAsAction="always"/>
    <item android:id="@+id/menu_fontSize"
        android:icon="@drawable/font_size"
        android:title="@string/menu_fontSize"
        android:showAsAction="ifRoom|withText" />
    <item android:id="@+id/menu_delete"
        android:icon="@drawable/ic_menu_delete"
        android:title="@string/menu_delete"
        android:showAsAction="ifRoom|withText" />
    <item android:id="@+id/voice"
        android:icon="@drawable/voice"
        android:title="@string/menu_voice"
        android:showAsAction="always">
    </item>

</menu>
```
## Screenshots:<br>
![This picture is stored in the folder of images.](https://github.com/huhuhuu/Mid-term-NotePad/blob/master/images/main.png)<br>
![This picture is stored in the folder of images.](https://github.com/huhuhuu/Mid-term-NotePad/blob/master/images/new.png)<br>
![This picture is stored in the folder of images.](https://github.com/huhuhuu/Mid-term-NotePad/blob/master/images/changeBackgroundColor.png)<br>
![This picture is stored in the folder of images.](https://github.com/huhuhuu/Mid-term-NotePad/blob/master/images/others.png)<br>
![This picture is stored in the folder of images.](https://github.com/huhuhuu/Mid-term-NotePad/blob/master/images/changeFontSize.png)<br>
![This picture is stored in the folder of images.](https://github.com/huhuhuu/Mid-term-NotePad/blob/master/images/voice.png)<br>
![This picture is stored in the folder of images.](https://github.com/huhuhuu/Mid-term-NotePad/blob/master/images/tips.png)<br>
![This picture is stored in the folder of images.](https://github.com/huhuhuu/Mid-term-NotePad/blob/master/images/changeTitle.png)<br>
![This picture is stored in the folder of images.](https://github.com/huhuhuu/Mid-term-NotePad/blob/master/images/changeTitle2.png)<br>
![This picture is stored in the folder of images.](https://github.com/huhuhuu/Mid-term-NotePad/blob/master/images/search.png)<br>
![This picture is stored in the folder of images.](https://github.com/huhuhuu/Mid-term-NotePad/blob/master/images/sort.png)<br>
![This picture is stored in the folder of images.](https://github.com/huhuhuu/Mid-term-NotePad/blob/master/images/parse.png)<br>








