---
layout: post
title:  "No Argument Flags in Spring-Shell"
author: "Greg"
category: cats-and-code
comments: true
---
![Ichabod the Cat](https://farm5.staticflickr.com/4236/35118897760_a6ca8f1cd5_c.jpg "Ichabod the Cat")

{% highlight java %}
@Component
public class SampleComponent implements CommandMarker {
    @CliCommand(value = {"remove-something"}, help = "Remove something")
    public String removeSomething(
        @CliOption(key = {"id"}, help = "The ID", mandatory = true)
            Long id,
        @CliOption(
            key = {"commit"},
            help = "Commit the transaction",
            unspecifiedDefaultValue = "dryrun",
            specifiedDefaultValue = "commit")
            String commit
    ) throws Exception {
        String result;
        if (commit.equals("commit")) {
            // commit flag present
        } else {
            // commit flag not present
        }
        return result;
    }
}
{% endhighlight %}
