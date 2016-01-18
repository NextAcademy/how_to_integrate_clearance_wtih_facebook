#Clearance
Clearance is a simple Ruby on Rails authentication with email & password.

#OmniAuth Facebook
OmniAuth Facebook is Facebook OAuth2 Strategy for OmniAuth

#Integrate OmniAuth Facebook with Clearance

To setup clearance, visit https://github.com/estelleccl/how_to_use_clearance or https://github.com/thoughtbot/clearance 

1. add ``` gem 'omniauth'``` into Gemfile

2. add ``` gem 'omniauth-facebook' ``` into Gemfile

3. run ``` bundle ``` in the terminal

4. create ```omniauth.rb```  file in ``` config\initializers ``` and paste the following code into the file:

	```
		Rails.application.config.middleware.use OmniAuth::Builder do
	 	  provider :facebook, 'APP_KEY', 'APP_SECRET'
		end
	```
	*replace 'APP_KEY' and 'APP_SECRET' with your own keys which you get in facebook
	
5. create ```authentication.rb```  file in ``` app\models ``` and paste the following code into the file:

	```
		class Authentication < ActiveRecord::Base
	
	 	 belongs_to :user
	
		  def self.create_with_omniauth(auth_hash)
		    create! do |auth|
		      auth.provider = auth_hash["provider"]
		      auth.uid = auth_hash["uid"]
		      auth.token = auth_hash["credentials"]["token"]
		    end
		  end
	
		  def update_token(auth_hash)
		    self.token = auth_hash["credentials"]["token"]
		    self.save
		  end
	
		end
	```

6. Go to ``` config\routes.rb``` and add the following code into the file:

	```
		get "/auth/:provider/callback" => "sessions#create_from_omniauth"
	```
7. create ```sessions_controller.rb```  file in ``` app\controllers ``` and paste the following code into the file:

	```
		class SessionsController < Clearance::SessionsController
	
		  def create_from_omniauth
		    auth_hash = request.env["omniauth.auth"]
	
		    authentication = Authentication.find_by_provider_and_uid(auth_hash["provider"], auth_hash["uid"]) || Authentication.create_with_omniauth(auth_hash)
		    if authentication.user
		      user = authentication.user 
		      authentication.update_token(auth_hash)
		      @next = root_url
		      @notice = "Signed in!"
		    else
		      user = User.create_with_auth_and_hash(authentication,auth_hash)
		      @next = edit_user_path(user)   
		      @notice = "User created - confirm or edit details..."
		    end
		    sign_in(user)
		    redirect_to @next, :notice => @notice
		  end
		end
	
	```

8. Go to ``` app/models/users``` and add the following code into the file:

	```
		has_many :authentications, :dependent => :destroy
	  
	  def self.create_with_auth_and_hash(authentication,auth_hash)
	    create! do |u|
	      u.first_name = auth_hash["info"]["first_name"]
	      u.email = auth_hash["extra"]["raw_info"]["email"]
	      u.authentications<<(authentication)
	    end
	  end
	
	  def fb_token
	    x = self.authentications.where(:provider => :facebook).first
	    return x.token unless x.nil?
	  end
	  
	  def password_optional?
	    true
	  end
	
	```
9. run ``` rails g migration create_authentications uid token provider user_id``` in the terminal

10. go to ``` db/migrate``` and find ```<timestamps_create_authentications.rb ``` and add the following code into the file:

	```
		t.timestamps null: false
	```

11. run ``` bundle exec rake db:migrate  ``` in the terminal

	```
		make sure authentications table with uid, token, provider columns has been created
	```

12. append the following code in ``` config/routes.rb````

	``` 	
		resources :users, only: [:show, :edit, :update, :destroy] 
	``` 
		*make sure there is no duplicates in the routes

#References
1. https://github.com/thoughtbot/clearance
2. https://github.com/estelleccl/how_to_use_clearance
3. https://gist.github.com/stevebourne/2394427

#Errors and solutions
1. https://github.com/estelleccl/how_to_integrate_clearance_wtih_facebook/blob/master/errors_and_solutions.md
