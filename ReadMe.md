# ADS04 Android

## 수업 내용

- Fragment와 ViewPager를 사용해 TabLayout을 만드는 학습

## Code Review

### MainActivity

```Java
public class MainActivity extends AppCompatActivity {

    ViewPager viewPager;
    TabLayout tabLayout;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        setTabLayout();
        setViewPager();

        setListener();
    }

    private void setListener(){
        // 탭레이아웃을 ViewPager와 연결
        tabLayout.addOnTabSelectedListener(
                new TabLayout.ViewPagerOnTabSelectedListener(viewPager)
        );
        // ViewPager의 변경사항을 탭레이아웃에 전달
        viewPager.addOnPageChangeListener(
                new TabLayout.TabLayoutOnPageChangeListener(tabLayout)
        );
    }

    private void setTabLayout(){
        // 탭 레이아웃 생성
        tabLayout = (TabLayout) findViewById(R.id.tabLayout);
        tabLayout.addTab( tabLayout.newTab().setText("One") );
        tabLayout.addTab( tabLayout.newTab().setText("Two") );
        tabLayout.addTab( tabLayout.newTab().setText("Three") );
        tabLayout.addTab( tabLayout.newTab().setText("Four") );

    }

    private void setViewPager(){
        viewPager = (ViewPager) findViewById(R.id.viewPager);
        CustomAdapter adapter = new CustomAdapter(getSupportFragmentManager());
        viewPager.setAdapter(adapter);
    }
}
```
- context가 아닌 getSupportFragmentManager()를 넘겨 준다.

### CustomAdapter

```Java
public class CustomAdapter extends FragmentStatePagerAdapter{
    private static final int COUNT = 4;
//    List<Fragment> data;

    public CustomAdapter(FragmentManager fm) {
        super(fm);
    }

    @Override
    public Fragment getItem(int position) {
        switch(position){
            case 1:
                return new TwoFragment();
            case 2:
                return new ThreeFragment();
            case 3:
                return new FourFragment();
            default:
                return new OneFragment();
        }
    }

    @Override
    public int getCount() {
        return COUNT;
    }

}
```

- 생성자에서 FragmentManager를 받는다.

### OneFragment

```Java
public class OneFragment extends Fragment {


    public OneFragment() {
        // Required empty public constructor
    }


    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.fragment_one, container, false);
    }

}
```
- 이하 two,three,four fragment는 동일하므로 생략

## 보충설명

### 특징

- Fragment를 ViewPager형식으로 보여줄때 통상적으로 FragmentPagerAdapter나 FragmentStatePagerAdapter를 사용
- 상속받는 구조를 보면 FragmentPagerAdapter를 FragmentStatePagerAdapter가 상속을 받고 있음.

### FragmentPagerAdapter

- FragmentManager에 등록하여둔 Fragment를 최대한 영구적으로 가지도록 구현되어 있음
- 적은양의 Fragment들을 사용할 경우에 사용하길 바람
- View가 Invisible되는 상황에서 Destory가 되더라도 Fragment들은 영구적으로 메모리에 남아 있게됨
- 페이지 수가 정해져 있고 그 수가 많지 않다면 FragmentPagerAdapter를 권장

#### + 심화보충

- 한 번 생성되면 Fragment의 인스턴스를 FragmentManager에서 절대로 제거하지 않기 때문(Activity가 종료되지 않는한)
- 재 보이지 않는 Fragment에서 View들을 detach한다. 당신의 Fragment가 범위 밖으로 나가면 onDestoryView()가 호출되고, 나중에 이 Fragment로 돌아가면 onCreateView()가 호출 될 것
- FragmentPagerAdapter를 사용할 때는 반드시 onDestroyView()에서 현재 View 또는 Context에 대한 참조를 제거해야 한다
-adapter에 묶인 작은 list의 항목들을 끊임없이 변경하면 메모리에 수백개의 detach된 인스턴스들을 가질 수 있다

### FragmentStatePagerAdapter

- 다소 많은 Fragment를 사용할 경우에 추천됨.
- Fragment들이 사용되지 않을 경우에는 state를 타게 되며, Destory가 되어버림.
- Fragment를 동적으로 Crreat/Destory하게되어서, 페이지 전환시 Overhead가 발생함.
- Fragment를 동적으로 붙였다가 띠었다가 할 경우에는 FragmentStatePagerAdapter를 사용하는 것을 권장
- 동적으로 사용하는 방법은 getItemPotision메소드를 오버라이드할 경우에, return POSITION_NONE을 한다.


#### + 심화보충

범위 밖의 Fragment 인스턴스를 FragmentManager에서 완전히 제거
Fragment 인스턴스는 당신이 다시 돌아왔을 때 재생성되고 상태는 복원

### notifyDataSetChanged() Issue

 notifyDataSetChanged() 메소드는 현재 표시되고 있는 Fragment나 그들의 view들을 갱신하기 위한 용도가 아니다. 만약 일부 데이터가 변경되어 그들의 뷰들을 갱신하고자 한다면 당신의 Fragment에 Listener/Callback을 추가

### 출처

- 출처 : http://mrgamza.tistory.com/215 [@Override]
- 출처 : http://eyeahs.github.io/blog/2016/12/06/fragmentstatepageradapter/

## TODO

- Frament를 사용할 떄의 생명주기에 따른 문제, 메모리 문제, 눈에 보이지 않고 계속 살아있는 인스턴스 문제 등 생각보다 많은 부분을 고려해야 한다고 하는데, 글로 봐선 잘 이해가 가지 않음으로, 직접 예제를 만들어 보면서, 오류를 만들어보고 상태를 체크해볼 것.

- 프래그먼트를 쓸때와 뷰를 쓸때와의 차이점 분석해보기

## Retrospect

- 프래그먼트를 사용하려면 제대로 알고 사용을 해야 할 것 같다. 
- 뷰페이저안의 프래그먼트의 대체안으로 뷰를 사용할 수 있다고 하는데, 코드가 어떻게 다르고, 뭐가 더 편한지 고려해 보기.

## Output
- 참고 : [ViewPagerWithFragment](http://grandbig.github.io/blog/2016/01/30/android-tablayout/)

![ViewPagerWithFragment](http://grandbig.github.io/images/android-tablayout.png)