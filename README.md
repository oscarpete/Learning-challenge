# Title: Test Driven Development

- Repository: `php-tdd`
- Type of Challenge: `Learning challenge`
- Duration: `3 days`
- Team challenge : `solo`

## Learning objectives
- Ability to write and read unit tests
- Understanding the importance of Test Driven Development

## The Mission
In the following scenario we are going to explore "Unit Tests" and Test Driven Development, feel free to ask your coach more information about this.
Start with watching this [great youtube introduction](https://www.youtube.com/watch?v=WMqe0jkqPMQ) to the subject.

We are going to create a simple booking software for meeting rooms.
You can write this in Symfony or in vanilla PHP, whatever you find most simple.

### What are unit tests?
Unit testing is testing small pieces of your code in isolation with test code. So instead of going to your browser and verifying everything works, you create a piece of code that checks if another piece of code works.

The immediate advantages that come to mind are:

- Running the tests becomes automate-able and repeatable
- You can test at a much more granular level than point-and-click testing via a GUI
- Once a test is written to prevent a certain bug, this bug can never happen again, improving long term stability.

### What is Test Driven Development?
Another way to look at unit testing is that you write the tests first. This is known as Test-Driven Development (TDD for short). TDD brings additional advantages:

- You don't write speculative "I might need this in the future" code -- just enough to make the tests pass
- The code you've written is always covered by tests
- By writing the test first, you're forced into thinking about how you want to call the code, which usually improves the design of the code in the long run.

### What is PHPUnit?
[PHPUnit](https://phpunit.de/) is the PHP version of the [xUnit architecture](https://en.wikipedia.org/wiki/XUnit) for unit testing frameworks. This means that many other languages have their own version of this unit testing framework. This means you will be able to write tests in many languages after learning about PHPUnit!

### Installation
#### Not using composer, not using symfony
Follow [the steps on the official site](https://phpunit.readthedocs.io/en/9.3/installation.html).

#### I am not using Symfony
Run `composer require --dev phpunit/phpunit ^9`

Check if it works with

`./vendor/bin/phpunit --version`

Always place your tests in the `tests/` directory, you will need to create this directory yourself.
After installation, run `./vendor/bin/phpunit tests`, this will run all valid tests in your tests directory.

#### I am using symfony
Rename the `phpunit.xml.dist` on the root to `phpunit.xml`.

Always place your tests in the `tests/` directory.
Now run `bin/phpunit`, this will run all valid tests in your tests directory.
***The first time you run this script this will also install PHPunit for you!***

## Must-have features
Create the following entities
 - User
    - password, email (if working with the login)
    - username OR email field (you can choose)
    - credit (integer, start credit 100)
    - premiumMember (bool, default false)
- Room
    - name
    - onlyForPremiumMembers (bool, default false)
- Bookings
    - Relation to room & User
    - Start date (datetime)
    - End date (datetime)
    
### General flow
For now just create rooms directly in the db, you do not need to provide an interface for this.

On the homepage the user gets to see all the rooms, with a link to book a room.
He then selects a start and end date and time between which he wants access to the room.
He is then charged 2 EUR for each hour he booked the room.

The following conditions apply:

 - Rooms marked as premium can only be hired for premium members
 - No room can be book for more than 4 hours
 - Check if they can afford the rent for the room
 - Room can only be booked if no other User has already booked it in this time (this is the most difficult condition)
 
***For all these conditions try to use Test Driven Development first.***

Let's do the first requirement together!

"Rooms marked as premium can only be hired for premium members"

First I am going to write my tests, without even writing any real application code!
I will obviously need both a User, and a Room object. 
I decided it makes the most sense if the function to check room availability is on the room object.

***The code below expects the constructor of Room & User to require a boolean to set their premium status. ***

So I create a new class called RoomAvailabilityTest inside the `tests/` directory.

```php 
//class has to end with Test
class CheckRoomAvailabilityTest extends TestCase
{
    /**
     * function has to start with Test
     */
    public function testPremiumRoom(): void
        $room = new Room(false);
        $user = new User(false);

        $this->assertTrue($room->canBook($user));
    }
}
```

At this point the test will of course fail, because we don't even have a function `canBook()` in the Room class.
So let us create this with some simple logic to pass the first test:

```php 
class Room {
    function canBook(User $user) {
        return true;
    }
}
```

While the test will succeed now, of course this is not that useful! The function always returns true for now.
Let us create a new test to make sure we check both conditions (fail & success).

```php 
//class has to end with Test
class CheckRoomAvailabilityTest extends TestCase
{
    /**
     * function has to start with Test
     */
    public function testPremiumRoom(): void
        $room = new Room(false);
        $user = new User(false);

        $this->assertTrue($room->canBook($user));

        $room = new Room(true);//premium room, with no premium user
        $user = new User(false);

        $this->assertFalse($room->canBook($user));
    }
}
```

```php 
class Room {
    function canBook(User $user) {
        return ($room->isPremium() && $user->isPremium()) || !$room->isPremium();
    }
}
```

Ending some more use cases in our tests, and we end up with this end result:

```php 
//class has to end with Test
class CheckRoomAvailabilityTest extends TestCase
{
    private function dataProviderForPremiumRoom() : array
    {
        return [
            [true, true, true],
            [false, false, true],
            [false, true, true],
            [true, false, false]
        ];
    }

    /**
     * function has to start with Test
     * @dataProvider dataProviderForPremiumRoom
     */
    public function testPremiumRoom(bool $roomVar, bool $userVar, bool $expectedOutput): void

        $room = new Room($roomVar);
        $user = new User($userVar);

        $this->assertEquals($expectedOutput, $room->canBook($user));
    }
}
```

## Nice to have
- Provide a page where the user can recharge his credit. Write a unit test for this!
- Create an admin role that can manage the rooms. In symfony you could use the `make:crud` command for this.
- Create a "password forget" flow. In symfony you could use the maker bundle for this.
