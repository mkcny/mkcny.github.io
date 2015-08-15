---
layout: post
title:  "Pre-Commit Hooks Done Right"
date:   2015-08-14
categories: git
---
There are a lot of pages out there that can show you how to set up a pre-commit hook to do some sanity checks before allowing you to commit changes, but there’s something I’ve seen people get wrong a lot of the time.

A common use case for a pre-commit hook is to do something like a syntax check. A really simple example for PHP would look something like this:

{% highlight bash %}
#!/bin/bash

# This is the wrong way!

set -e

php_files=$(git diff --cached --name-only --diff-filter=ACMRTUXB | grep "\.php$")

for file in $php_files
do
  php -l $file
done
{% endhighlight %}


The `--diff-filter` option includes everything but deleted files.

The problem with this example is that it doesn’t validate the syntax of the file in the state it will be committed. It validates the syntax of the current working copy version. Consider the following example:

{% highlight diff %}
$ git diff --staged
diff --git a/index.php b/index.php
new file mode 100644
index 0000000..af34a18
--- /dev/null
+++ b/index.php
@@ -0,0 +1,3 @@
+<?php
+
+0
{% endhighlight %}

What we've staged above contains a syntax error, but the working copy diff may include a fix:

{% highlight diff %}
$ git diff
diff --git a/index.php b/index.php
index af34a18..77d3284 100644
--- a/index.php
+++ b/index.php
@@ -1,3 +1,4 @@
 <?php

 0
+;
{% endhighlight %}

The pre-commit hook will allow this to pass even though what would be committed is a file with a syntax error. This is a contrived example, but it's not hard to imagine how this could come into play in the real world.

Thankfully, git has a command that will allow you to get the as-staged version of a file. A more correct pre-commit hook could run the syntax check like so:


{% highlight bash %}
git show :$file | php -l
{% endhighlight %}

With that in place, the staged changes in the earlier example will be correctly rejected.
