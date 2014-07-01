---
layout: page
title: Commands about Pixpalace[1]
tag: [ Rails, Console, Authentification, Commands, Mysql, SQL, Statistic ]
---
{% include JB/setup %}

##A continuer
*controllers/search_stats_controller.rb:19* SearchStatsController#create




### *Simulate Login with Authlogic in rails console*
*You must activate the Authlogic::Session::Base.controller with a controller object before creating objects*  

	> Authlogic::Session::Base.controller = Authlogic::ControllerAdapters::RailsAdapter.new(self)
	
	//Create a new session
	> newsession = UserSession.new(:login => "superadmin", :password => "fr4eo560")
	
	> newsession.save
	> current_user = newsession && newsession.user
	
	//Login out
	> newsession.destroy
	> current_user.destroy
	
	
---

	eval("@object.xyz")
	@object.send('xyz')
	
	x.y.z.attr
	
	x.send(y).send(z).send(attr)
	
	class MyClass
	  def [](method)
	    send(method)
	  end
	end
	
	x[y][z][attr]
	
	
	
	
---
### Mysql command

### *show the volume of databases*
	> select concat(round(sum(DATA_LENGTH/1024/1024), 2), 'MB') as data from TABLES;
	
	> select concat(round(sum(DATA_LENGTH/1024/1024), 2), 'MB') as data from TABLES where table_schema='forexpert';
	
	> select concat(round(sum(DATA_LENGTH/1024/1024),2),'MB') as data from TABLES where table_schema='forexpert' and table_name='member';


### *current used database*
	> select database();
	
### *Show indexes for tables*
	> show indexes from table_name;

---
### Specifying rails version to use when creating a new application

	$ rails _2.1.0_ new myapp 
	
	// Don't change the database
	$ rails console --sandbox
---

	$ ps -ef | grep _proc
	$ ls -ltr
	$ bundle exec rake assets:precompile
	