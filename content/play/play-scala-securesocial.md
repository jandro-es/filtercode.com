Title: Play framework and SecureSocial module.
Date: 2014-09-21 10:20
Category: Play
Tags: play, scala, securesocial, opengraph
Slug: play-scala-securesocial
Authors: jandro_es
Summary: Using **SecureSocial** authentication module in **Play Framework's versions 2.3.x**

Since Play's version **2.3.x** the subsystem to load, configure and use modules has changed. Now instead of loading [SecureSocial module](http://securesocial.ws) in a *play.plugins* file inside your application configuration directory, you need to load and initialise it in a global application class.

First we need to set up a new application using Activator's console, we'll choose **play-scala** templates and after some magic we'll have a fully functional Scala/Play application.

Now we need to tell **SBT** to load the SecureSocial module, we edit the file **build.sbt** and we add *ws.securesocial* module and the resolver needed to load the snapshot. It should look something like this:

~~~~{.language-scala}
	name := """demo-securesocial"""

	version := "1.0-SNAPSHOT"

	lazy val root = (project in file(".")).enablePlugins(PlayScala)

	scalaVersion := "2.11.1"

	resolvers += "Sonatype Snapshots" at "https://oss.sonatype.org/content/repositories/snapshots/"

	libraryDependencies ++= Seq(
  		jdbc,
  		anorm,
  		cache,
  		ws,
  		"ws.securesocial" %% "securesocial" % "master-SNAPSHOT"
  		)
~~~~

Reload your project and you'll have SecureSocial module ready to configure and use.

Following [SecureSocial's documentation](http://securesocial.ws/guide/user-service.html) we need to provide a custom implementation of the **UserService**. But first we need to define our User's model. We create a package called **models** inside the **app** package. Here we'll hold all our models. We create a simple *case class* for storing our users, we can evolve it later. We call the case class **DemoUser**:

~~~~{.language-scala}
	package models

	import securesocial.core._

	case class DemoUser(main: BasicProfile, identities: List[BasicProfile])

~~~~

Now we create another package inside the *app* package and we call it **services**, here we'll keep different services alongside the User's one. We create a new Scala class called **DemoUserService** with the following code. Note that we link all the user related actions to our User's model.

~~~~{.language-scala}
	package services

	import play.api.{Logger, Application}
	import securesocial.core._
	import securesocial.core.providers.{UsernamePasswordProvider, MailToken}
	import scala.concurrent.Future
	import securesocial.core.services.{UserService, SaveMode}
	import models.DemoUser
	import org.joda.time.format.{DateTimeFormatter, DateTimeFormat}

	class DemoUserService extends UserService[DemoUser] {
	  val logger = Logger("application.controllers.DemoUserService")

	  var users = Map[(String, String), DemoUser]()
	  //private var identities = Map[String, BasicProfile]()
	  private var tokens = Map[String, MailToken]()

	  def find(providerId: String, userId: String): Future[Option[BasicProfile]] = {
	    if (logger.isDebugEnabled) {
	      logger.debug("users = %s".format(users))
	    }
	    val result = for (
	      user <- users.values;
	      basicProfile <- user.identities.find(su => su.providerId == providerId && su.userId == userId)
	    ) yield {
	      basicProfile
	    }
	    Future.successful(result.headOption)
	  }

	  def findByEmailAndProvider(email: String, providerId: String): Future[Option[BasicProfile]] = {
	    if (logger.isDebugEnabled) {
	      logger.debug("users = %s".format(users))
	    }
	    val someEmail = Some(email)
	    val result = for (
	      user <- users.values;
	      basicProfile <- user.identities.find(su => su.providerId == providerId && su.email == someEmail)
	    ) yield {
	      basicProfile
	    }
	    Future.successful(result.headOption)
	  }

	  def save(user: BasicProfile, mode: SaveMode): Future[DemoUser] = {
	    mode match {
	      case SaveMode.SignUp =>
	        val newUser = DemoUser(user, List(user))
	        users = users + ((user.providerId, user.userId) -> newUser)
	      case SaveMode.LoggedIn =>

	    }
	    // first see if there is a user with this BasicProfile already.
	    val maybeUser = users.find {
	      case (key, value) if value.identities.exists(su => su.providerId == user.providerId && su.userId == user.userId) => true
	      case _ => false
	    }
	    maybeUser match {
	      case Some(existingUser) =>
	        val identities = existingUser._2.identities
	        val updatedList = identities.patch(identities.indexWhere(i => i.providerId == user.providerId && i.userId == user.userId), Seq(user), 1)
	        val updatedUser = existingUser._2.copy(identities = updatedList)
	        users = users + (existingUser._1 -> updatedUser)
	        Future.successful(updatedUser)

	      case None =>
	        val newUser = DemoUser(user, List(user))
	        users = users + ((user.providerId, user.userId) -> newUser)
	        Future.successful(newUser)
	    }
	  }

	  def link(current: DemoUser, to: BasicProfile): Future[DemoUser] = {
	    if (current.identities.exists(i => i.providerId == to.providerId && i.userId == to.userId)) {
	      Future.successful(current)
	    } else {
	      val added = to :: current.identities
	      val updatedUser = current.copy(identities = added)
	      users = users + ((current.main.providerId, current.main.userId) -> updatedUser)
	      Future.successful(updatedUser)
	    }
	  }

	  def saveToken(token: MailToken): Future[MailToken] = {
	    Future.successful {
	      tokens += (token.uuid -> token)
	      token
	    }
	  }

	  def findToken(token: String): Future[Option[MailToken]] = {
	    Future.successful {
	      tokens.get(token)
	    }
	  }

	  def deleteToken(uuid: String): Future[Option[MailToken]] = {
	    Future.successful {
	      tokens.get(uuid) match {
	        case Some(token) =>
	          tokens -= uuid
	          Some(token)
	        case None => None
	      }
	    }
	  }

	  def deleteExpiredTokens() {
	    tokens = tokens.filter(!_._2.isExpired)
	  }

	  override def updatePasswordInfo(user: DemoUser, info: PasswordInfo): Future[Option[BasicProfile]] = {
	    Future.successful {
	      for (
	        found <- users.values.find(_ == user);
	        identityWithPasswordInfo <- found.identities.find(_.providerId == UsernamePasswordProvider.UsernamePassword)
	      ) yield {
	        val idx = found.identities.indexOf(identityWithPasswordInfo)
	        val updated = identityWithPasswordInfo.copy(passwordInfo = Some(info))
	        val updatedIdentities = found.identities.patch(idx, Seq(updated), 1)
	        found.copy(identities = updatedIdentities)
	        updated
	      }
	    }
	  }

	  override def passwordInfoFor(user: DemoUser): Future[Option[PasswordInfo]] = {
	    Future.successful {
	      for (
	        found <- users.values.find(_ == user);
	        identityWithPasswordInfo <- found.identities.find(_.providerId == UsernamePasswordProvider.UsernamePassword)
	      ) yield {
	        identityWithPasswordInfo.passwordInfo.get
	      }
	    }
	  }
	}
~~~~

As you can see in the code, for all the response we use **Futures** so the application will be fully *reactive*-

The next step is to let the application know to use our User's model and it's matching User's service. In order to accomplish this, we need to extend the **Runtime Environment** of the application.

Inside our **app** package we create a new **Global** object which extends Play's global settings, we call it, wait for it..., **Global**:

~~~~{.language-scala}
	import java.lang.reflect.Constructor
	import securesocial.core.RuntimeEnvironment
	import securesocial.core.providers._
	import securesocial.core.providers.utils.{Mailer, PasswordHasher, PasswordValidator}
	import services.{DemoUserService}
	import models.DemoUser
	import scala.collection.immutable.ListMap

	object Global extends play.api.GlobalSettings {

	}
~~~~

now we extend the application's runtime environment:

~~~~{.language-scala}
	import java.lang.reflect.Constructor
	import securesocial.core.RuntimeEnvironment
	import securesocial.core.providers._
	import securesocial.core.providers.utils.{Mailer, PasswordHasher, PasswordValidator}
	import services.{DemoUserService}
	import models.DemoUser
	import scala.collection.immutable.ListMap

	object Global extends play.api.GlobalSettings {

	  /**
	   * Demo application's custom Runtime Environment
	   */
	  object DemoRuntimeEnvironment extends RuntimeEnvironment.Default[DemoUser] {
	    override lazy val userService: DemoUserService = new DemoUserService
	    override lazy val providers = ListMap(
	      include(new FacebookProvider(routes, cacheService, oauth2ClientFor(FacebookProvider.Facebook))),
	      include(new GitHubProvider(routes, cacheService, oauth2ClientFor(GitHubProvider.GitHub))),
	      include(new GoogleProvider(routes, cacheService, oauth2ClientFor(GoogleProvider.Google))),
	      include(new LinkedInProvider(routes, cacheService, oauth1ClientFor(LinkedInProvider.LinkedIn))),
	      include(new TwitterProvider(routes, cacheService, oauth1ClientFor(TwitterProvider.Twitter))),
	      include(new UsernamePasswordProvider[DemoUser](userService, avatarService, viewTemplates, passwordHashers))
	    )
	  }

	}
~~~~

You can see in the code that we had also added the allowed providers, in this instance we use Facebook, Github, Google, LinkedIn, Twitter and the standard login/password one. For the entire available list you can check SecureSocial's documentation.

As you probably know, **Play Framework** don't have any **Dependency Injection** system out of the box, so you can pick up the one that suits you. The following code can be replaced with any Dependency Injection library you want, for a simple use the following code using a *Cake pattern* will be enough.

~~~~{.language-scala}
	import java.lang.reflect.Constructor
	import securesocial.core.RuntimeEnvironment
	import securesocial.core.providers._
	import securesocial.core.providers.utils.{Mailer, PasswordHasher, PasswordValidator}
	import services.{DemoUserService}
	import models.DemoUser
	import scala.collection.immutable.ListMap

	object Global extends play.api.GlobalSettings {

	  /**
	   * Demo application's custom Runtime Environment
	   */
	  object DemoRuntimeEnvironment extends RuntimeEnvironment.Default[DemoUser] {
	    override lazy val userService: DemoUserService = new DemoUserService
	    override lazy val providers = ListMap(
	      include(new FacebookProvider(routes, cacheService, oauth2ClientFor(FacebookProvider.Facebook))),
	      include(new GitHubProvider(routes, cacheService, oauth2ClientFor(GitHubProvider.GitHub))),
	      include(new GoogleProvider(routes, cacheService, oauth2ClientFor(GoogleProvider.Google))),
	      include(new LinkedInProvider(routes, cacheService, oauth1ClientFor(LinkedInProvider.LinkedIn))),
	      include(new TwitterProvider(routes, cacheService, oauth1ClientFor(TwitterProvider.Twitter))),
	      include(new UsernamePasswordProvider[DemoUser](userService, avatarService, viewTemplates, passwordHashers))
	    )
	  }

	   /**
	   * Dependency injection on Controllers using Cake Pattern
	   *
	   * @param controllerClass
	   * @tparam A
	   * @return
	   */
	  override def getControllerInstance[A](controllerClass: Class[A]): A = {
	    val instance = controllerClass.getConstructors.find { c =>
	      val params = c.getParameterTypes
	      params.length == 1 && params(0) == classOf[RuntimeEnvironment[DemoUser]]
	    }.map {
	      _.asInstanceOf[Constructor[A]].newInstance(DemoRuntimeEnvironment)
	    }
	    instance.getOrElse(super.getControllerInstance(controllerClass))
	  }

	}
~~~~

Now we need to configure the SecureSocial module, to do this, we edit the **application.conf** file and at the end we add the following two lines:

~~~~{.language-scala}
	include "smtp.conf"
	include "securesocial.conf"
~~~~

We create two files at the same package level as **application.conf** one called *smtp.conf* (we need to send mail for the Username/Password provider) and the other one called **securesocial.conf**.

We include in **smtp.conf** our emailing configuration, i recomned to have a look at [Mandril](http://mandrill.com) it's an amazing mailing provider with a really simple configuration. The file will look something like this:

~~~~{.language-scala}
	#####################################################################################
	#
	# SMTP Settings
	#
	#####################################################################################
	smtp {
	  host = smtp.mandrillapp.com
	  port = 587
	  ssl = true
	  user = "<mandriluser>"
	  password = <mandril_password>
	  from = "<mail_address>"
	}
~~~~

and our **securesocial.conf** file:

~~~~{.language-scala}
	#####################################################################################
	#
	# SecureSocial 2 Settings
	#
	#####################################################################################
	securesocial {
	  ssl = false
	  cookie {
	    absoluteTimeOutInMinutes = 720
	  }

	  twitter {
	    requestTokenUrl = "https://twitter.com/oauth/request_token"
	    accessTokenUrl = "https://twitter.com/oauth/access_token"
	    authorizationUrl = "https://twitter.com/oauth/authenticate"
	    consumerKey = your_consumer_key
	    consumerSecret = your_consumer_secret
	  }

	  facebook {
	    authorizationUrl = "https://graph.facebook.com/oauth/authorize"
	    accessTokenUrl = "https://graph.facebook.com/oauth/access_token"
	    clientId = your_client_id
	    clientSecret = your_client_secret
	    # this scope is the minimum SecureSocial requires.  You can add more if required by your app.
	    scope = email
	  }

	  google {
	    authorizationUrl = "https://accounts.google.com/o/oauth2/auth"
	    accessTokenUrl = "https://accounts.google.com/o/oauth2/token"
	    clientId = your_client_id
	    clientSecret = your_client_secret
	    scope = "profile email"
	  }

	  linkedin {
	    requestTokenUrl = "https://api.linkedin.com/uas/oauth/requestToken"
	    accessTokenUrl = "https://api.linkedin.com/uas/oauth/accessToken"
	    authorizationUrl = "https://api.linkedin.com/uas/oauth/authenticate"
	    consumerKey = your_consumer_key
	    consumerSecret = your_consumer_secret
	  }

	  github {
	    authorizationUrl = "https://github.com/login/oauth/authorize"
	    accessTokenUrl = "https://github.com/login/oauth/access_token"
	    clientId = your_client_id
	    clientSecret = your_client_secret
	  }

	  userpass {
	    #
	    # Enable username support, otherwise SecureSocial will use the emails as user names
	    #
	    withUserNameSupport = false
	    sendWelcomeEmail = true
	    enableGravatarSupport = true
	    tokenDuration = 60
	    tokenDeleteInterval = 5
	    signupSkipLogin = false
	  }
	}
~~~~

you'll need to replace the values for the accounts created on the different providers. And as a last step in our configuration, we need to create the routes needed for authentication, the **routes** file should look like this:

~~~~{.language-scala}
	# Home page
	GET     /                           @controllers.Application.index

	# Map static resources from the /public folder to the /assets URL path
	GET     /assets/*file               controllers.Assets.at(path="/public", file)

	# Login page
	GET     /login                      @securesocial.controllers.LoginPage.login
	GET     /logout                     @securesocial.controllers.LoginPage.logout

	# User Registration and password handling
	GET     /signup                     @securesocial.controllers.Registration.startSignUp
	POST    /signup                     @securesocial.controllers.Registration.handleStartSignUp
	GET     /signup/:token              @securesocial.controllers.Registration.signUp(token)
	POST    /signup/:token              @securesocial.controllers.Registration.handleSignUp(token)
	GET     /password                   @securesocial.controllers.PasswordChange.page
	POST    /password                   @securesocial.controllers.PasswordChange.handlePasswordChange

	# Providers entry points
	GET     /authenticate/:provider     @securesocial.controllers.ProviderController.authenticate(provider)
	POST    /authenticate/:provider     @securesocial.controllers.ProviderController.authenticateByPost(provider)
~~~~

and with that, we already have our application configured for using SecureSocial and the ability to secure any action.

Let's secure our home page, we need to edit the **Application's** main controller, it should look like this:

~~~~{.language-scala}
	package controllers

	import play.api.mvc.{Action, RequestHeader}
	import securesocial.core._
	import models.DemoUser


	class Application(override implicit val env: RuntimeEnvironment[DemoUser]) extends securesocial.core.SecureSocial[DemoUser] {

	  def index = SecuredAction { implicit request =>
	    Ok(views.html.index("Your new application is ready."))
	  }
	}
~~~~

That's it, now when you compile the project and go to the home page, it will redirect you to the **Login** page.

**TIP :** If you're using **OSX Mavericks** or newer, you need to install **JAVA 1.7 SDK**, with the default 1.6 there are some incompatibility issues.

The full code of this example is available in [Github](https://github.com/jandro-es/demo-securesocial).