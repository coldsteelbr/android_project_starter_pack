# Android project starter pack
For mor information read the wiki page

## Base classes
_/base_ directory contains basic classes to implement different types of activities and patterns, such as list activity based on RecyclerView and View holder pattern. Also basic activities implement basic functionality to work with Dagger 2 (with inject() method).

### Getting started with basic activity

Extend **BaseActivity** class to implement simple empty activity and override abstract methods:

```java
public class MyActivity extends BaseActivity {
    
    ....
    // CONSTANTS
    private final static String CUSTOM_EXTRA_DATA = "CUSTOM_EXTRA_DATA";
    private final static String CUSTOM_EXTRA_STRING_DATA = "CUSTOM_EXTRA_STRING_DATA";
    
    //FIELDS
    @Inject
    Something mSomething;

	/** Static method starting new instance of activity */
    public static void showNewActivity(Context context, Long data, String stringData) {
        Intent intent = new Intent(context, MyActivity.class);
        intent.putExtra(CUSTOM_EXTRA_DATA, data);
        intent.putExtra(CUSTOM_EXTRA_STRING_DATA, stringData);
        context.startActivity(intent);
    }

	// SAMPLE IMPLEMENTATION OF THIS METHOD
    @Override
    public void inject() {
        getAppComponent().inject(this);
    }

    ....
}
```

### Getting started with basic LIST activity

**BaseRecyclerViewActivity** is based on the RecyclerView class. It, in turn, extends **BaseActivity**. Before creating your list activity, you must create a successor of base.views.holders.BaseViewHolder<T> class, where T represents the item model:

```java
    public class MyViewHolder extends BaseViewHolder<MyEntity>
        implements View.OnClickListener {
    // Widgets of a list item
    private TextView tvContent;
    private long mID;
    private Context mContext;
	// VIEW HOLDER NEED CONTEXT TO START OTHER ACTIVITIES ON CLICK
    public MyViewHolder(View itemView, Context context) {
        super(itemView);
        mContext = context;
        // finding subview of an item
        tvContent = itemView.findViewById(R.id.tv_content);
        itemView.setOnClickListener(this);
    }

	// Implementing item binding abstract method
    @Override
    public void bindItem(MyEntity entity) {
        mID = entity.getId();
        tvContent.setText(entity.getContent());
    }

	// As a bonus View Holder may be used as an item event listener.
	// Here we implement onClick listener, that starts an activty
	// by using class static method
    @Override
    public void onClick(View view) {
        MyActivity.showNewActivity(mContext, mID);
    }
}
```

Then create a layout file containing RecyclerView:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout ...>
	....
    <android.support.v7.widget.RecyclerView
        android:id="@+id/rv_my_list"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
	....
</LinearLayout>
```

Also create a layout file for an item:
**my_item.xml**
```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout ...>

    <LinearLayout ...>

        <TextView
            android:id="@+id/tv_content"
            .../>

    </LinearLayout>

    <!-- UNDERLINE (SPLITTER) -->
    <View
        android:layout_width="match_parent"
        android:layout_height="1dp"
        android:layout_gravity="bottom"
        android:background="@color/clItemUnderline"/>
</FrameLayout>
```


After that extend **BaseRecyclerViewActivity** to implement your list activity and override abstract methods:
```java
public class MyListActivity extends BaseRecyclerViewActivity<MyEntity, MyViewHolder> {
    // FIELDS
    @Inject
    Box<MyEntity> mMyEntityBox;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // DO NOT SET CONTENT VIEW !!!!
        //setContentView(R.layout.activity_training_session_list);
		....

    }

	// SET CONTENT VIEW BY IMPLEMENTING THIS METHOD
    @Override
    protected int getScreenLayout() {
        return R.layout.activity_my_list;
    }

	// UPDATE UI ON RESUME
    @Override
    protected void onResume() {
        super.onResume();
        updateUI();
    }

	// UPDATE UI EXAMPLE BASED ON OBJECT BOX
    private void updateUI() {
        // getting items and refreshing list
        Query<MyEntity> queryAll
                = mMyEntityBox.query()
                .build();

        List<MyEntity> filteredRecords = queryAll.find();

        refreshList(filteredRecords);
    }

	// RECYCLER VIEW ID FROM THE LAYOUT GOES HERE
    @Override
    protected int getRecyclerViewID() {
        return R.id.rv_my_list;
    }

	// SET ITEM LAYOUT ID
    @Override
    protected int getItemLayoutID() {
        return R.layout.my_item;
    }

	// CREATING NEW VIEW HOLDERS
    @Override
    protected MyViewHolder newViewHolder(View itemView) {
        return new MyViewHolder(itemView, this);
    }

	// INJECTION
    @Override
    public void inject() {
        getAppComponent().inject(this);
    }

    .....
    
}
```
