# IOT Thermostats Solution
**Develoment Environment**
* Ruby 2.6.2
* Rails 5.2.3
* PostgreSQL 9.5.15
* Redis 5.0.3

### How to run the application

First clone the repository to the local machine then cd into the project directory

Install gems
`bundle install`

Create Database
`bundle exec rake db:create`

Run the Migrations
`bundle exec rake db:migrate`

Seed the thermostats data
`bundle exec rake db:seed`

Start the Redis Server
`redis-server`

Now run the Test Suite
`bundle exec rspec`

Got a GREEN signal? Start up your server now.
`rails server`

Start Sidekiq
`sidekiq -C config/sidekiq.yml`

### API Endpoints Examples
In this project, JSON Web Token ( JWT ) Authentication has been implemented. So, before testing any endpoint, an `auth_token` needs to be generated by using thermostat `household_token`. Then, we have to pass that `auth_token` in the headers whatever API endpoint we are testing.

URL / ENDPOINT    |    VERB    |    DESCRIPTION   
----------------- | ---------- | -------------- 
/api/v1/thermostats/generate_auth_token       |    GET    | Generate Auth token
/api/v1/readings            |    POST    | Create a reading      
/api/v1/readings/{id}           |    GET     | Return a reading
/api/v1/thermostats/stats   |    GET   |   Return stats having avg, min and max values

 1. Generate auth token by using `household_token`

         curl -X GET \
          'http://localhost:3000/api/v1/thermostats/generate_auth_token?household_token=4016cf55-e199-44c9-a4a5-698a04ebf6ab'
     Expected Response
 
         {
           "thermostat": {
                "id": 1,
                "household_token": "4016cf55-e199-44c9-a4a5-698a04ebf6ab",
                "location": "5567 Kevin Estate",
                "created_at": "2019-04-19T11:06:35.652Z",
                "updated_at": "2019-04-19T11:06:35.652Z"
            },
            "auth_token": "eyJhbGciOiJIUzI1NiJ9.eyJob3VzZWhvbGRfdG9rZW4iOiI0MDE2Y2Y1NS1lMTk5LTQ0YzktYTRhNS02OThhMDRlYmY2YWIiLCJleHAiOjE1NTU5Mjk4NzN9.CFVWaKwruS0Lp-A6DMsqOeBrLBdI9YppJcoF1BRx3O8"
         }
    `auth_token` generated above needs to be passed in every request headers.

2. Create Reading - Data is persisted to the DB by sidekiq BG job but record is available from Redis DB immediately even if BG job is not completed.

         curl -X POST \
         'http://localhost:3000/api/v1/readings/?temperature=3.5&humidity=44&battery_charge=75' \
         -H 'Authorization: eyJhbGciOiJIUzI1NiJ9.eyJob3VzZWhvbGRfdG9rZW4iOiI0MDE2Y2Y1NS1lMTk5LTQ0YzktYTRhNS02OThhMDRlYmY2YWIiLCJleHAiOjE1NTU5Mjk4NzN9.CFVWaKwruS0Lp-A6DMsqOeBrLBdI9YppJcoF1BRx3O8'
     Expected Response
 
         {
           "reading": {
              "id": 1,
              "thermostat_id": 1,
              "seq_number": 1,
              "temperature": 3.5,
              "humidity": 44,
              "battery_charge": 75,
              "created_at": null,
              "updated_at": null
           }
         }

3. Get reading data. Reading ID needs to be given.

         curl -X GET \
          http://localhost:3000/api/v1/readings/1 \
         -H 'Authorization: eyJhbGciOiJIUzI1NiJ9.eyJob3VzZWhvbGRfdG9rZW4iOiI0MDE2Y2Y1NS1lMTk5LTQ0YzktYTRhNS02OThhMDRlYmY2YWIiLCJleHAiOjE1NTU5Mjk4NzN9.CFVWaKwruS0Lp-A6DMsqOeBrLBdI9YppJcoF1BRx3O8'

     Expected Response
 
         {
            "id": 1,
            "thermostat_id": 1,
            "seq_number": 1,
            "temperature": 3.5,
            "humidity": 44,
            "battery_charge": 75,
            "created_at": null,
            "updated_at": null
         }

4. Get thermostats data.

         curl -X GET \
         http://localhost:3000/api/v1/thermostats/stats \
         -H 'Authorization: eyJhbGciOiJIUzI1NiJ9.eyJob3VzZWhvbGRfdG9rZW4iOiI0MDE2Y2Y1NS1lMTk5LTQ0YzktYTRhNS02OThhMDRlYmY2YWIiLCJleHAiOjE1NTU5Mjk4NzN9.CFVWaKwruS0Lp-A6DMsqOeBrLBdI9YppJcoF1BRx3O8
     Expected Response
 
         {
           "temperature": {
               "avg": 3.15,
               "min": 6.3,
               "max": 6.3
            },
            "humidity": {
               "avg": 28,
               "min": 56,
               "max": 56
            },
            "battery_charge": {
                "avg": 23.5,
                "min": 47,
                "max": 47
            }
         }
    
  
