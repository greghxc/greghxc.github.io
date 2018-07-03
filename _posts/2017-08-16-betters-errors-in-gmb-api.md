---
layout: post
title:  "Better Errors Using the Google My Business API Java Client"
author: "Greg"
comments: true
---
![placeholder](https://farm5.staticflickr.com/4215/34664199694_abf41b2606_c.jpg "Large example image")

I wanted to do a thing, and maybe you do to! It goes something like this.
<!--more-->
{% highlight java %}
public static void main(String[] args) throws Exception {
    HttpTransport httpTransport = GoogleNetHttpTransport.newTrustedTransport();
    Credential credential = authorize();
    JsonFactory jsonFactory = JacksonFactory.getDefaultInstance();
    MyBusinessRequestInitializer initializer =
        new VerboseMyBusinessRequestInitializer();

    mybusiness = new Mybusiness.Builder(httpTransport, jsonFactory, credential)
        .setApplicationName("best-app-ever")
        .setMyBusinessRequestInitializer(initializer)
        .build();

    Mybusiness.Accounts.List accountsList = mybusiness.accounts().list();
    ListAccountsResponse response = accountsList.execute();
    List accounts = response.getAccounts();
}
{% endhighlight %}

{% highlight java %}
public class VerboseMyBusinessRequestInitializer
    extends MyBusinessRequestInitializer {

    public VerboseMyBusinessRequestInitializer() {
        super();
    }

    public VerboseMyBusinessRequestInitializer(String key) {
        super(key);
    }

    public VerboseMyBusinessRequestInitializer(String key, String userIp) {
        super(key, userIp);
    }

    protected void initializeMyBusinessRequest(MyBusinessRequest<?> request)
        throws IOException {

        HttpHeaders headers = request.getRequestHeaders();
        headers.set("X-GOOG-API-FORMAT-VERSION", 2);
        request.setRequestHeaders(headers);
    }
}
{% endhighlight %}
