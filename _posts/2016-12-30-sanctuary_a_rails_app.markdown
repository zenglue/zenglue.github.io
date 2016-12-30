---
layout: post
title:  "Sanctuary: a Rails app"
date:   2016-12-30 03:56:46 +0000
---

Continuing with my portfolio projects theme of different ways to engage migrant communities through web development, my Rails portfolio project centers around sanctuary cities. This web app allows users to upvote or downvote a city based on it's declared status as a sanctuary city or not, and leave a corresponding comment formed by their own experiences or perceptions of a city. With boundaries between state rights/local governance and federal enforcement constantly shifting on many different political issues, migration included, the intended purpose of this app is to give a clearer picture to users of the app of just how much of a sanctuary for migrants a city might be no matter what its officially declared status is. For example, a city with a declared status of sanctuary might be the target of more aggressive federal enforcement. There are definitely other ways to aggregate this data (i.e. deportation stats in a particular city) but crowdsourcing this information might be just as useful, albeit, subjective. Each upvote for a city is intended as the digital equivalent of a user putting a "Refugees are welcome" sign in their window. Safety as well as resources for migrants in any given community might be more of a reflection of its community rather than local or federal government mandates.

In making this app, my first major hurdle was how to model the data and to consider it's intended audience during this modeling. While voting on a city is not limited to migrants, it would be ironic to have migrants register and then persist in a database, but then how does the app track votes? I did not have a good solution around this and voting is limited to registered users. Currently, the code allows one vote per user by validating a :user_id and scoping its uniqueness to a
:city_id in the Vote model.

```
validates :user_id, uniqueness: {scope: :city_id}
```

One :user_id, one vote, otherwise the votes would be meaningless. On a related note, the site uses Devise for it's authentication methods and calls the Devise current_user method quite a bit in both the controllers and views. the current_user method relies on session data and finding a corresponding :id matched to the :id found in the database. current_user breaks if current_user is nil. Viewing the site as a guest user while using Devise is not possible (to my knowledge) without overriding Devise's current_user method. The code is located in my application_controller. I kept it sparse.

```
def current_user
    super || guest_user
  end

  def guest_user
    User.new(username: "guest")
  end
end
```

The current_user method uses Ruby's 'super' to call the method on itself or call guest_user and makes a new user in cache with the username: "guest" but never saves them to the database. Storing a guest_user :id in session data and having guest accounts persist in the database was a possibility, but I was reluctant to do so given the domain of the app (his would have allowed voting privileges, but potentially many stale accounts when this app inevitably goes viral).
The current system to upvote or downvote a city is currently implemented with button_to links. A link_to form seemed convoluted for something as simple as an upvote or downvote. However, having the vote methods in the city_controller seemed to be a separation of concerns issue and so they are located in the voting controller via a nested resource. I'll be honest, it's little hacky. The route table for this looks like:

```
  resources :cities do
    resources :votes do
      member do
        post 'upvote'
        post 'downvote'
      end
    end
  end
```

And the corresponding routes are:

```
upvote_city_vote POST     /cities/:city_id/votes/:id/upvote(.:format)   votes#upvote
downvote_city_vote POST     /cities/:city_id/votes/:id/downvote(.:format) votes#downvote
```

Can you spot the issue?
There's no params(:votes => :id) in the route since these are essentially non-RESTful create routes. The vote :id's don't exist before creation. Passing in @city to upvote_city_vote(@city), uses the same @city.id for both :id symbols in the route, but since it's a button_to and Rails doesn't complain about the invalid :id, I left the solution as is. Since button_to is a post method the route is never shown in the browser. It's worth noting that this bypasses Rails strong_params so it's potentially a security issue.

Making this web app was full of compromises. Not all of which I'm entirely happy with, but it left me with a lot to think about, not the least of which being just how expansive the Rails universe is.

	

