在Activity启动Fragment的时候可以进行传值。传值的方法有两种，两种方法都可以使用，只是更加推荐第二种。

1. 第一种是添加extra信息
跟activity启动activity的时候添加extra信息的方法是一样。这个方法的好处是代码简单，副作用是破坏了fragment的复用。因为extra信息是通过activity获取的。

创建fragment实例，并且加入extra信息
```java
public class CrimeActivity extends  SingleFragmentActivity {

    public static final String EXTRA_CRIME_ID = "com.bignerdranch.android.criminalintent.crime_id";

    public static Intent newIntent(Context packageContext, UUID crimeId) {
    Intent intent = new Intent(packageContext, CrimeActivity.class);
    //添加extra信息
    intent.putExtra(EXTRA_CRIME_ID, crimeId); 
    return intent; 
    } ...
}
```


获取extra信息
```java
public class CrimeFragment extends Fragment{
    ...
    public onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    mCrime = new Crime();

    //获取从activity传过来的extra信息
    //此方法种还需要获取托管fragment的activity的信息，因此依赖关系比较强
    UUID crimeId = (UUID)getActivity().getIntent() .getSerializableExtra(CrimeActivity.EXTRA_CRIME_ID);
    mCrime = CrimeLab.get(getActivity()).getCrime(crimeId);

} ...

}

```

2. 第二种方法是使用arguments bundle

每个Fragment实例都可以附带一个Bundle对象。该bundle包含键-值对。
```java
Bundle args = new Bundle(); 
args.putSerializable(ARG_MY_OBJECT, myObject); 
args.putInt(ARG_MY_INT, myInt); 
args.putCharSequence(ARG_MY_STRING, myString);
```

要把argument bundle附加给Fragment的方法是需要调用Fragment.setArguments(Bundle)。而且，还必须在创建fragment后，在添加给activity之前完成setArguments。

```java
public class CrimeFragment extends Fragment {

    private static final String ARG_CRIME_ID = "crime_id";

    private Crime mCrime; private EditText mTitleField; 
    private Button mDateButton; private CheckBox mSolvedCheckbox;

    public static CrimeFragment newInstance(UUID crimeId) { 
        Bundle args = new Bundle(); args.putSerializable(ARG_CRIME_ID, crimeId);

        CrimeFragment fragment = new CrimeFragment(); 
        fragment.setArguments(args); 
        return fragment;

} ...

}
```

使用newInstance的方法
```java
public class CrimeActivity extends SingleFragmentActivity {

    public private static final String EXTRA_CRIME_ID = "com.bignerdranch.android.criminalintent.crime_id"; ...

    @Override
    protected Fragment createFragment() { 
        return new CrimeFragment(); 
        UUID crimeId = (UUID)getIntent().getSerializableExtra(EXTRA_CRIME_ID);

        return CrimeFragment.newInstance(crimeId); 
    }

}
```

在fragment中获取argment的方法
```java
@Override
public void onCreate(Bundle savedInstanceState) {

    super.onCreate(savedInstanceState); 
    UUID crimeId = (UUID) getActivity().getIntent().getSerializableExtra(CrimeActivity.EXTRA_CRIME_ID); 
    UUID crimeId = (UUID) getArguments().getSerializable(ARG_CRIME_ID); 
    mCrime = CrimeLab.get(getActivity()).getCrime(crimeId);

}
```