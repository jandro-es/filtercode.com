Title: Avoid memory problems with Composer
Date: 2014-07-03 10:20
Category: DevOps
Tags: composer, memory, symfony, tip
Slug: avoid-memory-problems-composer
Authors: jandro_es
Summary: Sometimes when deploying a Symfony application or any other PHP application which uses **Composer** you'll find out that you ran out of memory. With this quick win you can prevent that to happen.

Sometimes when deploying a Symfony application or any other PHP application which uses **Composer** you'll find out that you ran out of memory when executing:

~~~~{.language-bash}
	composer update
~~~~

Instead of increase (you might even don't have anymore) the memory allocated to *composer*, a solution was pointed out in [Composer's Github Channel](https://github.com/composer/composer/issues).

The solution is to upload to the repository the **composer.lock** file created in you local environment. Remove **composer.lock** from your **gitignore** file, if present, and in your development machine execute:

~~~~{.language-bash}
	composer update
~~~~

add it to your **git** repository

~~~~{.language-bash}
	git add composer.lock
~~~~

and change you deployment script to execute 

~~~~{.language-bash}
	composer install
~~~~

instead of

~~~~{.language-bash}
	composer update
~~~~

Now, aside of not running out of memory, your deployment should execute faster.


