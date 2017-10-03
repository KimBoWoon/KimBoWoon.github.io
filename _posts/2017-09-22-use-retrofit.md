---
title: "[Android] Retrofit 사용법"
date: 2017-09-22 01:23:45
categories:
- Android
---

# Android Retrofit 사용법
Retrofit은 Square Open Source에서 만든 REST API 통신을 위한 안드로이드 라이브러리이다. 안드로이드 서비스를 개발하려면 서버와의 통신은 꼭 필요합니다. 다른 라이브러리를 사용하면 불필요하고 귀찮은 작업이 많습니다. Retrofit은 불필요한 작업을 최소화 하고 쓰기 쉽게 만들어진 라이브러리입니다.

먼저 gradle을 추가합니다.
```gradle
compile 'io.reactivex.rxjava2:rxandroid:2.0.1'
compile 'io.reactivex.rxjava2:rxjava:2.1.3'
compile 'com.squareup.retrofit2:retrofit:2.3.0'
compile 'com.google.code.gson:gson:2.7'
compile 'com.android.support:recyclerview-v7:25.1.0'
compile 'com.squareup.retrofit2:converter-gson:2.1.0'
compile 'com.squareup.picasso:picasso:2.5.2'
```

간단하게 파싱할 https://randomuser.me/api의 데이터입니다.

```json
{
    results: [
        {
            gender: "male",
            name: {
                title: "mr",
                first: "jimi",
                last: "polon"
            },
            location: {
                street: "9350 itsenäisyydenkatu",
                city: "kristinestad",
                state: "south karelia",
                postcode: 14156
            },
            email: "jimi.polon@example.com",
            login: {
                username: "blackdog726",
                password: "ranger",
                salt: "cFE2HMuR",
                md5: "c7caf7af79bc56b4ddeee96bb9f90d85",
                sha1: "c2a0382305915dc106e2b75a48b2f454aef4f334",
                sha256: "ff6fbdd32246bd365b3a095c91408503a1f73620526e4f24da4759f439427096"
            },
            dob: "1967-10-03 16:02:57",
            registered: "2009-07-19 00:26:27",
            phone: "02-657-116",
            cell: "042-656-81-18",
            id: {
                name: "HETU",
                value: "1167-163D"
            },
            picture: {
                large: "https://randomuser.me/api/portraits/men/76.jpg",
                medium: "https://randomuser.me/api/portraits/med/men/76.jpg",
                thumbnail: "https://randomuser.me/api/portraits/thumb/men/76.jpg"
            },
            nat: "FI"
        }
    ],
    ...
    info: {
    seed: "468e4162245cdf64",
    results: 1,
    page: 1,
    version: "1.1"
    }
}
```

일단 데이터를 얻어오기 위해 interface를 정의합니다.
```java
public interface APIInterface {
    @GET("/api/")
    Call<PersonModel> getUsers(@Query("results") int results);
}
```
실제로 요청하는 url은 https://randomuser.me/api/?results=1 입니다. 여기 result에 따라 몇 개의 데이터를 얻어올지 결정합니다.
그리고 인터페이스에 있는 메소드의 인자들은 url에 있는 ? 뒤에 query를 설명합니다.

model도 적절하게 생성합니다.
```java
class Name {
    @SerializedName("title")
    @Expose
    private String title;
    @SerializedName("first")
    @Expose
    private String first;
    @SerializedName("last")
    @Expose
    private String last;

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getFirst() {
        return first;
    }

    public void setFirst(String first) {
        this.first = first;
    }

    public String getLast() {
        return last;
    }

    public void setLast(String last) {
        this.last = last;
    }
}

class UserProfileImage {
    @SerializedName("large")
    @Expose
    private String large;
    @SerializedName("medium")
    @Expose
    private String medium;
    @SerializedName("thumbnail")
    @Expose
    private String thumbnail;

    public String getLarge() {
        return large;
    }

    public void setLarge(String large) {
        this.large = large;
    }

    public String getMedium() {
        return medium;
    }

    public void setMedium(String medium) {
        this.medium = medium;
    }

    public String getThumbnail() {
        return thumbnail;
    }

    public void setThumbnail(String thumbnail) {
        this.thumbnail = thumbnail;
    }
}

class Item {
    @SerializedName("name")
    @Expose
    private Name name;
    @SerializedName("picture")
    @Expose
    private UserProfileImage userProfileImage;
    @SerializedName("gender")
    @Expose
    private String gender;
    @SerializedName("phone")
    @Expose
    private String phone;

    public String getGender() {
        return gender;
    }

    public void setGender(String gender) {
        this.gender = gender;
    }

    public String getPhone() {
        return phone;
    }

    public void setPhone(String phone) {
        this.phone = phone;
    }

    public Name getName() {
        return name;
    }

    public void setName(Name name) {
        this.name = name;
    }

    public UserProfileImage getUserProfileImage() {
        return userProfileImage;
    }

    public void setUserProfileImage(UserProfileImage userProfileImage) {
        this.userProfileImage = userProfileImage;
    }
}

public class PersonModel {
    @SerializedName("results")
    @Expose
    private List<Item> items = null;
    @SerializedName("page")
    @Expose
    private int page;

    public List<Item> getItems() {
        return items;
    }

    public void setItems(List<Item> items) {
        this.items = items;
    }

    public int getPage() {
        return page;
    }

    public void setPage(int page) {
        this.page = page;
    }
}
```

또 결과를 보여줄 recyclerview도 만듭니다.
```java
interface ItemClickListener {
    void onItemClick(int position);
}

public class RecyclerAdapter extends RecyclerView.Adapter<RecyclerView.ViewHolder> implements ItemClickListener {
    private List<Item> items;
    private Context context;

    public RecyclerAdapter(Context context, List<Item> items) {
        this.context = context;
        this.items = items;
    }

    public String getItemTitle(int position) {
        return items.get(position).getName().getFirst();
    }

    @Override
    public void onItemClick(int position) {
        Toast.makeText(context, getItemTitle(position), Toast.LENGTH_SHORT).show();
    }

    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        RecyclerView.ViewHolder holder = null;

        View v = LayoutInflater.from(parent.getContext()).inflate(R.layout.list_item, parent, false);
        holder = new UsersViewHolder(v, this);

        return holder;
    }

    @Override
    public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
        if (holder == null) {
            return;
        }

        ((UsersViewHolder) holder).name.setText(items.get(position).getName().getFirst());
        Picasso.with(context).load(items.get(position).getUserProfileImage().getThumbnail()).into(((UsersViewHolder) holder).profileImg);
    }

    @Override
    public int getItemCount() {
        return this.items.size();
    }

    static class UsersViewHolder extends RecyclerView.ViewHolder {
        private TextView name;
        private ImageView profileImg;

        public UsersViewHolder(View itemView, final ItemClickListener itemClickListener) {
            super(itemView);
            this.name = (TextView) itemView.findViewById(R.id.text);
            this.profileImg = (ImageView) itemView.findViewById(R.id.userImg);

            itemView.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    itemClickListener.onItemClick(getAdapterPosition());
                }
            });
        }
    }
}
```

마지막으로 retrofit을 만듭니다.
```java
Retrofit client = new Retrofit.Builder().baseUrl("https://randomuser.me").addConverterFactory(GsonConverterFactory.create()).build();
APIInterface service = client.create(APIInterface.class);
Call<PersonModel> call = service.getUsers(10);
call.enqueue(new Callback<PersonModel>() {
    @Override
    public void onResponse(Call<PersonModel> call, Response<PersonModel> response) {
        if (response.isSuccessful()) {
            Log.i("Success", response.toString());
            PersonModel person = response.body();
            recyclerView.setAdapter(new RecyclerAdapter(getApplicationContext(), response.body().getItems()));
        }
    }

    @Override
    public void onFailure(Call<PersonModel> call, Throwable t) {
        Log.i("Failed", "Failed Connection");
    }
});
```
