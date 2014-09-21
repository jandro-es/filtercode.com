Title: Create mixins/mapped superclasses in Doctrine.
Date: 2014-08-20 10:20
Category: Symfony
Tags: symfony, database, doctrine, opengraph
Slug: mixins-doctrine-abstract
Authors: jandro_es
Summary: Using abstract classes, create mixins/mapped superclasses in **Doctrine** and keep using doctrine migrations.

Most often than not you need a series of entities to have a number of fields that cannot be denormalized on another table and maybe you need some listener/observer that acts on entities with that same fields. You could use **reflection** to accomplish this, but it's quite messy to use. There is a simple way, we can create those entities using **mixins** or in *Doctrine* lingo **Mapped superclasses**.

Let's suppose you have two entities, **Articles** and **Videos** both in need of **OpenGraph** data. If we create a **mixin** (a.k.a. abstract mapped class) called *OGEntity* defined as follows:

~~~~{.language-php}
	/**
 	* @ORM\MappedSuperclass
 	*/
	abstract class BaseOGEntity 
	{
		/**
	     * @ORM\Column(name="og_name",type="string", nullable=true)
	     */
	    private $og_name;

	    /**
	     * @ORM\Column(name="og_text",type="text", nullable=true)
	     */
	    private $og_text;

	    /**
	     * @ORM\Column(name="og_image",type="string", nullable=true)
	     */
	    private $og_image;

	    /**
	     * @ORM\Column(name="title",type="string", nullable=false)
	     */
	    private $title;

	    /**
	     * @ORM\Column(name="summary",type="text", nullable=false)
	     */
	    private $summary;

	    /**
	     * @ORM\Column(name="image",type="string", nullable=false)
	     */
	    private $image;
	}

~~~~

It's important to declare the members of the abstract class as **private** and inside the annotation "@ORM" to declare the **name** filed, besides not mandatory, the *Doctrine migrations* command will not work properly without it. Of course you'll need to provide as well the matching getters and setters.

Now we can define our entities much more simpler and with code reutilisation:

**Articles Entity**
~~~~{.language-php}
	/**
	 * @ORM\Table(name="articles")
	 */
	class Articles extends OGEntity
	{
		/**
	     * @ORM\Id
	     * @ORM\Column(type="integer")
	     * @ORM\GeneratedValue(strategy="IDENTITY")
	     */
	    protected $id;

	    /**
	     * @ORM\Column(type="text", nullable=true)
	     */
	    protected $text;
	}
~~~~

**Videos Entity**
~~~~{.language-php}
	/**
	 * @ORM\Table(name="videos")
	 */
	class Videos extends OGEntity
	{
		/**
	     * @ORM\Id
	     * @ORM\Column(type="integer")
	     * @ORM\GeneratedValue(strategy="IDENTITY")
	     */
	    protected $id;

	    /**
	     * @ORM\Column(type="text", nullable=true)
	     */
	    protected $embed;
	}
~~~~

Now when we execute the commands:

~~~~{.language-bash}
	php app/console doctrine:migrations:diff
~~~~

and 
~~~~{.language-bash}
	php app/console doctrine:migrations:execute
~~~~

we'll see that our database's tables *articles* and *videos* already have rows called *og_name, og_text, og_image, title, summary, image*.

Another advantage of these **mixins** is to have a certain *type casting*, imagine that the *OpenGraph* fields aren't editable by the user, but composed based on what the user enters on the *title, summary and image* fields. We'll setup a listener on the **postPersist** and **postUpdate** events for all entities and in the listener we will discriminate like this:

~~~~{.language-php}
	 public function postPersist(LifecycleEventArgs $args)
	 {
	 	if ($entity instanceof OGEntity)
	 	{
	 		if (NULL === $entity->getOgName()) {
            	$entity->setOgName($entity->getTitle());
        	}
	        if (NULL === $entity->getOgText() === NULL) {
	            $entity->setOgText($entity->getSummary());
	        }
	        if (NULL === $entity->getOgImage()) {
	            $entity->setOgImage($entity->getImage());
	        }
	 	} 
	}
~~~~

This way we'll always have up to date our OpenGraph values.