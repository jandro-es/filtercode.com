Title: Avoid memory problems with Composer
Date: 2014-07-03 10:20
Category: DevOps
Tags: composer, memory, symfony, tip
Slug: avoid-memory-problems-composer
Authors: jandro_es
Summary: Sometimes when you try to deploy a **Symfony** application or other PHP application using **Composer** on a Virtual Machine/Environment without much RAM memory (dedicated or allocated to PHP) you'll find that most of the times composer fails due to **exhausting** **memory**. With this quick win you can prevent that to happen.

Sometimes when you try to deploy a **Symfony** application or other PHP application using **Composer** on a Virtual Machine/Environment without much RAM memory (dedicated or allocated to PHP) you'll find that most of the times composer fails due to **exhausting** **memory**.

One solution, as mentioned on Github's Composer channel is to submit to your repository the composer.lock file and run install instead of update. The workflow will be as follows:

On your development environment remove **composer.lock** from the **gitignore** file if present.

Update your lock file with:

~~~~{.language-markup}
	composer update
~~~~

And finally on your server/deployment script instead of executing

~~~~{.language-markup}
	composer update
~~~~

execute

~~~~{.language-markup}
	composer install
~~~~

You'll notice that even on machines with as few as 512MB of RAM you'll be able to use Composer without memory problems.
