---
title: Main Test Atom Editor
layout: post
date: 2018-11-19 01:18
tags: [editor markdown]
---

# heading 1
## heading 2
1. wertwertewtr

{% mermaid %}
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->D;
{% endmermaid %}


{% highlight ruby linenos %}
def show
  @widget = Widget(params[:id])
  respond_to do |format|
    format.html # show.html.erb
    format.json { render json: @widget }
  end
end
{% endhighlight %}