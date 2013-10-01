---
layout: post
title: "ActiveRecord like callbacks using active_model"
date: 2013-10-01 10:40
comments: true
published: false
categories:
---


**ActiveRecord like callbacks using active_model**

We have seen ActiveRecord callbacks which are executed before or after the object is created or updated.
But how do we provide the similar interface for none persistent models. Well here ActiveModel callbacks comes to rescue.
ActiveModel Callbacks provides an interface for any class to have Active Record like callbacks.
Similar to active record, the callback chain is aborted as soon as one of the methods in the chain returns false.

First of all we will need to require active_model
<pre>
require 'active_model'
</pre>
then extend any ruby class with
<pre>
class Transaction
  extend ::ActiveModel::Callbacks
end
</pre>
Then we need to define a list of methods that we want callbacks attached to using define_model_callbacks
<pre>
define_model_callbacks :withdraw, :deposit
</pre>
Here **:withdraw** and **:deposit** are the methods where we want to attach callbacks

Defining callback methods using **define_model_callbacks** allows us to define methods which can be executed **before, after and around** the model methods ie:; withdraw and deposit methods

This will provide all three standard callbacks (before, around and after) for both the :withdraw and :deposit methods. **To implement, we wrap the methods we want callbacks on in a block so that the callbacks get a chance to fire:**
<pre>
def withdraw
  run_callbacks :withdraw do
    # Our withdraw code here
  end
end

def deposit
  run_callbacks :deposit do
    # Our deposit code here
  end
end
</pre>

**run_callbacks** tells to run callbacks along with the main model method.

Now we can define which method to execute when (before, after or around) the model method is invoked in the following way.
<pre>
before_withdraw :check_if_authentic_user
after_deposit :send_confirmation
</pre>
So we define **check_if_authentic_user** method and this method gets executed before withdraw method gets executed.
Similarly **send_confirmation** method gets executed after deposit method gets executed.

So our class our overall code becomes
<pre>
require 'active_model'

class Transaction
  extend ::ActiveModel::Callbacks

  define_model_callbacks :withdraw, :deposit

  before_withdraw :check_if_authentic_user
  after_deposit :send_confirmation

  def withdraw
    run_callbacks :withdraw do
      puts "Money withdrawn"
    end
  end

  def deposit
    run_callbacks :deposit do
      puts "deposited"
    end
  end

  def check_if_authentic_user
    puts "Checking if user is authentic"
  end

  def send_confirmation
    puts "Confirmation sent"
  end

end


transaction = Transaction.new
transaction.withdraw
transaction.deposit

Outputs

Checking if user is authentic
Money withdrawn
deposited
Confirmation sent
</pre>

**But if we return false from the callback method then the callback chain gets terminated.**<br/>

For example replace <i>check_if_authentic_user</i> with this.
<pre>
def check_if_authentic_user
  puts "Authenticating user..."
  false
end
</pre>

Our output would be
<pre>
Authenticating user...
deposited
Confirmation sent
</pre>
Since callback method **check_if_authentic_user** returned false the main **:withdraw** method never gets executed
