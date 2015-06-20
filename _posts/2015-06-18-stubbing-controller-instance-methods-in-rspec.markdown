---
layout: post
title: "How to Stub Controller Methods in RSpec Request Specs! (FTW)"
date: 2015-06-18
categories: RSpec
---

Recently I've had to restructure our [company's][fitplan] API, and part of that was writing new request specs for everything.

One <strong>HUGE</strong> headache I ran into while writing request specs for the API was bypassing controller `before_actions` such as `require_user`, as well as stubbing out crucial controller instance methods like `current_user`.
<br><br>

I tried searching online; I don't exaggerate when I say that I tried nearly everything:

{% highlight ruby %}
let(:user) { FactoryGirl.create(:user) }

controller.stub(:current_user) { user }
#=> deprecated syntax

allow(controller).to receive(:current_user) { user }
#=> couldn't recognize current_user

allow(controller).to receive(:current_user).and_return(user) 
#=> still couldn't recognize current_user...

allow(ApplicationController)to receive(:current_user).and_return(user)
#=> by this point I was ready to give up...
{% endhighlight %}

You get the idea... Hours of searching Stack Overflow and Relish and different docs didn't yield me the answer I needed.
<br><br>

Then I realized that `current_user`, `require_login` and all these methods were <strong>INSTANCE METHODS</strong>. Not class methods. <strong>Here's the correct way to stub out instance methods in RSpec:</strong>

{% highlight ruby %}
let(:user) { FactoryGirl.create(:user) }

it "tests a controller action that requires a current user" do
  allow_any_instance_of(ApplicationController).to receive(:current_user).and_return(user)
  get "/some/controller/action/path"

  expect(response.code).to eq("200")
  # ...
end
{% endhighlight %}

This RSpec helper allows you to stub out any instance method that exists on any class. However, my major use for this has been for stubbing out controller methods such as `current_user`, any controller before actions and <strong>access tokens</strong> if I'm doing request specs on a Rails API.
<br><br>

<em>Example of controller action to test:</em>

{% highlight ruby %}
# Here we have users#show which will 
# render the occupation of the user in json

class Api::UsersController < Api::ApplicationController
  doorkeeper_for :all
  before_action: require_user

  def show
    occupation = current_user.occupation
    render json: { occupation: occupation }, status: 200
  end

end
{% endhighlight %}
<br>

In this example, we are using the [Doorkeeper gem][doorkeeper] to protect the controller against unauthorized API calls (the following example will work with other token authorization methods too, the process is similar).

So in this situation we would have to stub out the access token and the current user in order to successfully test the response of our request:

{% highlight ruby %}
describe "GET /users/show " do
  let(:user) { FactoryGirl.create(:user) }
  let(:token) { double acceptable?: true } # this is way to stub doorkeeper tokens, if you're using other gems or methods, please refer to their respective counterparts.

  before(:each) do
    allow_any_instance_of(API::ApplicationController).to receive(:doorkeeper_token).and_return(token)
    allow_any_instance_of(API::ApplicationController).to receive(:current_user).and_return(user)
    get "/users/show"
  end

  it "should respond with status 200" do
    expect(response.code).to eq("200")
  end
end

#=> 1 example, 1 success
{% endhighlight %}

We have now succesfully stubbed out `current_user` and `doorkeeper_token` and can now successfully test our `controller#action`!
<br><br>

<strong>BUT WAIT.</strong> There's a bunch of code used just for stubbing, it would suck if we had to repeat that code over and over again for every request spec that we create... So let's move that into a spec support in order to write less code!

{% highlight ruby %}
# /app/spec/support/api_controller_helper.rb

module ApiControllerHelper
  def stub_access_token(token)
    allow_any_instance_of(Api::ApplicationController).to receive(:doorkeeper_token).and_return(token)
  end

  def stub_current_user(user)
    allow_any_instance_of(Api::ApplicationController).to receive(:current_user).and_return(user)
  end
end

RSpec.configure |config| do
  config.include ApiControllerHelper
end
{% endhighlight %}
<br>

In the end, our spec will look like this:

{% highlight ruby %}
describe "GET /users/show " do
  let(:user) { FactoryGirl.create(:user) }
  let(:token) { double acceptable?: true }

  before(:each) do
    stub_access_token(token)
    stub_current_user(user)
    get "/users/show"
  end

  it "should respond with status 200" do
    expect(response.code).to eq("200")
  end
end

#=> 1 example, 1 success
{% endhighlight %}
<br>

It took me quite a while to figure this out but in the end it was more than worth it. Armed with this knowledge, I'm sure you'll be able to bang out pages and pages of request specs in no time!
<br><br>

<strong>TL;DR</strong> Use `allow_any_instance_of(Object).to receive(method_on_object).and_return(mock_method_used_for_testing)` to mock an instance method on any object. Such as `current_user` on a controller.
<br><br><br>

<h3><strong>Fuck yeah.</strong></h3>



[fitplan]: http://fitplan.io
[doorkeeper]: https://github.com/doorkeeper-gem/doorkeeper