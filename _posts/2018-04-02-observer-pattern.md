# Observer Pattern
어떤 클래스에 변화가 있을 때 변화를 알려주는 패턴

# 
```
interface Observer {
    void update(String title, String news);
}

interface Publisher {
    void add(Observer observer);
    void delete(Observer observer);
    void notifyObserver();
}

class NewsMachine implements Publisher {
    private ArrayList<Observer> observers;
    private String title;
    private String news;

    public NewsMachine() {
        observers = new ArrayList<Observer>();
    }

    @Override
    public void add(Observer observer) {
        observers.add(observer);
    }

    @Override
    public void delete(Observer observer) {
        int index = observers.indexOf(observer);
        observers.remove(index);
    }

    @Override
    public void notifyObserver() {
        for (Observer observer : observers) {
            observer.update(title, news);
        }
    }

    public void setNewsInfo(String title, String news) {
        this.title = title;
        this.news = news;
        notifyObserver();
    }

    public String getTitle() {
        return title;
    }

    public String getNews() {
        return news;
    }
}

class AnnualSubscriber implements Observer {
    private String newsString;
    private Publisher publisher;

    public AnnualSubscriber(Publisher publisher) {
        this.publisher = publisher;
        publisher.add(this);
    }

    @Override
    public void update(String title, String news) {
        this.newsString = title + "\n" + news + "\n";
        display();
    }

    public void display() {
        System.out.println("annual user : today news " + newsString);
    }
}

class EventSubscriber implements Observer {
    private String newsString;
    private Publisher publisher;

    public EventSubscriber(Publisher publisher) {
        this.publisher = publisher;
        publisher.add(this);
    }

    @Override
    public void update(String title, String news) {
        this.newsString = title + "\n" + news + "\n";
        display();
    }

    public void withdraw() {
        publisher.delete(this);
    }

    public void display() {
        System.out.println("event user : today news " + newsString);
    }
}

public class ObserverPattern {
    public static void main(String[] args) {
        NewsMachine newsMachine = new NewsMachine();
        AnnualSubscriber as = new AnnualSubscriber(newsMachine);
        EventSubscriber es = new EventSubscriber(newsMachine);

        newsMachine.setNewsInfo("오늘 뉴스", "화창한 날씨");
        newsMachine.setNewsInfo("오늘 온도", "소풍가기 좋은 날");
    }
}
```

# 참조
* http://flowarc.tistory.com/entry/%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4-%EC%98%B5%EC%A0%80%EB%B2%84-%ED%8C%A8%ED%84%B4Observer-Pattern
