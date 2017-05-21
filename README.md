#  NotePad
------
## This is an AndroidStudio rebuild of google SDK sample NotePad
## 基本功能：
#### 时间戳的实现：
![时间戳]()
##### insert()函数中，每次新插入数据时，通过调用GetTime.Get_Now_Time_Long()函数获取当前时间；然后再待插入的Values中将CREATE_DATE，MODIFICATION_DATE两列的数值替换成当前时间。主要修改代码如下：
	// 得到当前时间
	Long now = GetTime.Get_Now_Time_Long();
	
	//初始化一条新纪录时，需要把当前时间插入
	if (values.containsKey(NotePad.Notes.COLUMN_NAME_CREATE_DATE) == false) {
	values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE, now);
	}
	
	//初始化一条新纪录时，需要把当前时间插入
	if (values.containsKey(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE) == false) {
	values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, now);
	}
##### update()函数中，每次更新时，更新的Values中的MODIFICATION_DATE数据也会一并更新到当前时间。
	//获取当前时间
	Long now = GetTime.Get_Now_Time_Long();
	//放入更新的Value中
	values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,now);

#### 搜索功能的实现，模糊查询标题：
![搜索]()

#####首先是NoteList中onOptionsItemSelected(MenuItem item) onOptionsItemSelected()：方法中，新增加了menu_Search选项，并且调用customView()方法导入布局文件 代码如下
	@Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
        case R.id.menu_add:
           startActivity(new Intent(Intent.ACTION_INSERT, getIntent().getData()));
	   return true;
	case R.id.menu_paste:
          startActivity(new Intent(Intent.ACTION_PASTE, getIntent().getData()));
          return true;
        case R.id.menu_Search://如果选择的是查找
                //这里直接使用Dialog弹出对话框进行查询输入
                System.out.println("NotesList—上下文菜单点击选项—menu_Search");

              customView();//新增方法-引用布局文件
                return true;
        default:
            return super.onOptionsItemSelected(item);
       }
    }
##### 新增方法：customView()：用于导入布局文件R.layout.search_by_title作为弹出对话框的样式；同时根据用户选择的取消/确认进行事件响应；如果选择确认，这使用Search()方法进行查询。 代码如下：
	public void customView( )
    {
        TableLayout search_view = (TableLayout)getLayoutInflater().inflate(
                R.layout.search_by_title,null
       );
        //这里一定要用上面哪行代码的 search_view.findViewById
        // 不能用this.findViewById因为没有加载 search_by_title.xml 找不到R.id.SearchTitle，
        //结果之后的mText回事NUll
       //但是也不能用setContentView(R.layout.search_by_title) 还是会报错(xml)
       mText = (EditText) search_view.findViewById(R.id.SearchTitle);

	        new AlertDialog.Builder(this)
               .setTitle("查询框标题")
               .setView(search_view)
              .setPositiveButton("确定", new DialogInterface.OnClickListener() {//设置取消按钮
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        //创建一个·Toast信息
                        Toast.makeText(NotesList.this,"你选择了确认查询！！",
                                //设置显示的时间
                                Toast.LENGTH_SHORT).show();
                        Search();
                   }
                })
               .setNegativeButton("取消", new DialogInterface.OnClickListener() {//设置取消按钮
	                   @Override
                   public void onClick(DialogInterface dialog, int which) {
                       //创建一个·Toast信息
	                        Toast.makeText(NotesList.this,"你选择了取消查询！！",
	                 //设置显示的时间
                                Toast.LENGTH_SHORT).show();

 	               }
	            })
	            .create()
             .show();
  }
##### Search()：用于偶去用户输入的查询关键字并且进行数据库的查询，返回mCursor，如果返回值非空，再调用refresh()刷新，重新加载当前页面List项。 代码如下：
	protected void Search() {

    //默认URI即为整个表
    mUri = NotePad.Notes.CONTENT_URI;//这里先试试notes的URI



    String Title="";//Title默认初始化
    //这里是点击EditTitle时
    //把输入框中需要查找的数据

    if(  mText!=null )//以防万一可能的NULLPointException
        if(mText.getText() != null)
            Title = mText.getText().toString();//读取


    if( mText==null )System.out.println("mText 是 null！！");


    System.out.println("Title"+Title);





    //查询条件语句
    String selection =  NotePad.Notes.COLUMN_NAME_TITLE + " like ? ";
    //查询条件语句的条件值
    String[]   selectionArgs = {"%" + Title + "%"};
    System.out.println("AA!!!==" + selectionArgs[0]);
    System.out.println("MURI==" + mUri);

    mCursor=getContentResolver().query(
            mUri,    // The URI for the note to update.
            PROJECTION,  // The values map containing the columns to update and the values to use.
            selection,    // 查询条件 -where titile = ??
            selectionArgs,     // 查询条件的值  ??的值
            null       //排序
    );//查询数据完成



    if (mCursor != null) {//不为空才星星刷新

        System.out.println("刷新页面");
        refresh();
    }
    else
        System.out.println("刷新页面失败 mCursor==NULL");

}
##### refresh()：根据查询的结果mCursor，重新加载当前页面List项。 代码如下：
	public void refresh() {
	    String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE,
	            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE} ;
	
	    int[] viewIDs = { android.R.id.text1 ,android.R.id.text2};//显示的组件数量要等于dataColumns   一对一关系
	
	
	    // 用SimpleCursorAdapter加载查询结果
	
	    SimpleCursorAdapter adapter
	            = new SimpleCursorAdapter(
	            this,                             // The Context for the ListView
	            R.layout.noteslist_item,          //表明列表项的布局
	            mCursor,                           // 表示从查询得到的光标中的记录作为适配器的数据
	            dataColumns,                      //显示的内容
	            viewIDs                           //显示用的视图
	    );
	    // Sets the ListView's adapter to be the cursor adapter that was just created.
	    setListAdapter(adapter);
	}
##### 修改布局文件：list_options_menu.xml：添加了查询选项 代码如下：
		<item android:id="@+id/menu_Search"
	       android:icon="@drawable/ic_menu_compose"
	       android:title="查询"
	        android:alphabeticShortcut='p' />
		</menu>
	##### 新增布局文件：search_by_title.xml：作为弹出的查询对话框的布局样式。 代码如下：
		<?xml version="1.0" encoding="utf-8"?>
	<TableLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:layout_width="wrap_content"
	    android:layout_height="wrap_content"
	    android:id="@+id/Tablelayout"
	    android:paddingLeft="6dip"
	    android:paddingRight="6dip"
	    android:paddingBottom="3dip">
	
	<!--本文件是按标题查询的弹出窗口布局文件-->
	
	    <TableRow android:layout_width="match_parent"
	        android:layout_height="wrap_content">
	
	    <!--用户名输入框-->
	    <EditText android:id="@+id/SearchTitle"
	        android:layout_width="match_parent"
	        android:layout_height="wrap_content"
	        android:hint="请输入查询111"
	        android:selectAllOnFocus="true"/>
	
	   </TableRow>
	</TableLayout>
