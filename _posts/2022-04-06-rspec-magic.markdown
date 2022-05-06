---
layout: post
author: Phillip
title: "RSpec: Let's Describe Context"
date: 2022-04-06 16:37:27 -0600
categories: Ruby Rspec
---

<h1>RSpec Magic</h1>
RSpec is a popular Ruby testing framework that provides a lot of powerful features for you to poke and prod your apps to get them ready for production. Developers can learn the basic structure of tests fairly quickly but the magic-like syntax can be confusing, especially to early learners. So let’s dive into some of the building blocks of a spec.

The Domain-Specific Language (DSL) of RSpec feels magical at first. The syntax and structure creates very readable lines and blocks that are almost sentence like. This is in part from RSpec embracing Ruby's implicit style. You can easily leave out lots of parenthesis and use lots of one-liners to create small, readable tests but they may not be clear what is happening.

```ruby
require 'rspec'
require './lib/bank_account'

RSpec.describe BankAccount do
  describe 'can be created with default balance' do
    it { is_expected.to be_instance_of BankAccount }
    it { is_expected.to have_attributes funds: 0 }
  end

  context 'with available funds' do
    let(:account) { BankAccount.new(30) }
    it 'can withdraw funds' do
      expect { account.withdraw_funds(20) }.to change { account.funds }.from(30).to(10)
    end
  end

  context 'with no available funds' do
    let(:account) { BankAccount.new(0) }
    it 'cannot withdraw funds' do
      expect { account.withdraw_funds(30) }.not_to change { account.funds }
    end
  end
end
```

So let’s break down this test and look at what each block and method is doing. First by adding in parentheses and expanding the curly brace one-liners to do/end blocks, it become more clear what parts are methods, arguments, and blocks

```ruby
require('rspec')
require('./lib/bank_account')

RSpec.describe(BankAccount) do
  describe('can be created with default balance') do
    it('is a bank account') do
       is_expected.to(be_instance_of(BankAccount))
     end
    it('has zero default funds') do
       is_expected.to(have_attributes({funds: 0 }))
     end
  end

  context('with available funds') do
    let(:account) { BankAccount.new(30) }
    it('can withdraw funds') do
      expect { account.withdraw_funds(20) }.to change { account.funds }.from(30).to(10)
    end
  end

  context('with no available funds') do
    let(:account) { BankAccount.new(0) }
    it('cannot withdraw funds') do
      expect { account.withdraw_funds(30) }.not_to change { account.funds }
    end
  end
end
```

The parentheses make it more clear what is an argument being passed to a method. So in line 4
`RSpec.describe(BankAccount)` shows us that `RSpec` is a class that we are calling the `describe` method on and we are passing it an argument of `BankAccount`, which is the Ruby class that we are testing.

Everything in the rest of the test is inside of the do and end for this code block, so all the rest of the code is being called within the scope of `Rspec.describe`.

<h3>Describe and Context</h3>

Now looking at line 5 we see the method `describe` being called. Then on line 14 and line 21 we see similar code blocks calling the method `context`. These are both methods given to us by RSpec (which we have access to since we are within the scope of `RSpec` from line 4). `describe` and `context` are aliases for the same method, they are interchangeable. They are both used to group related assertions together in logical chunks. This is a great example of RSpec embracing Ruby’s philosophy of giving developers several ways to do the same thing, allowing for developer choice. We could use only `describe` or only `context` and the tests will run the same way

```ruby
require('rspec')
require('./lib/bank_account')

RSpec.context(BankAccount) do
  context('can be created with default balance') do
    it('is a bank account') do
       is_expected.to(be_instance_of(BankAccount))
     end
    it('has zero default funds') do
       is_expected.to(have_attributes({funds: 0 }))
     end
  end

  context('with available funds') do
    let(:account) { BankAccount.new(30) }
    it('can withdraw funds') do
      expect { account.withdraw_funds(20) }.to change { account.funds }.from(30).to(10)
    end
  end

  context('with no available funds') do
    let(:account) { BankAccount.new(0) }
    it('cannot withdraw funds') do
      expect { account.withdraw_funds(30) }.not_to change { account.funds }
    end
  end
end
```
```ruby
require('rspec')
require('./lib/bank_account')

RSpec.describe(BankAccount) do
  describe('can be created with default balance') do
    it('is a bank account') do
       is_expected.to(be_instance_of(BankAccount))
     end
    it('has zero default funds') do
       is_expected.to(have_attributes({funds: 0 }))
     end
  end

  describe('with available funds') do
    let(:account) { BankAccount.new(30) }
    it('can withdraw funds') do
      expect { account.withdraw_funds(20) }.to change { account.funds }.from(30).to(10)
    end
  end

  describe('with no available funds') do
    let(:account) { BankAccount.new(0) }
    it('cannot withdraw funds') do
      expect { account.withdraw_funds(30) }.not_to change { account.funds }
    end
  end
end
```

So the choice is yours. In practice developers often use `describe` to describe a thing and use `context` to outline different scenarios, or provide context. So in line 4 we describe the class we are testing and then line 14 and line 21 we use context to outline two different scenarios that we expect to have different outcomes.

Note at one time there was one difference, `context` could not be used as a top level method only describe could, that is where we are calling `RSpec.describe` in line 4. Change log here. This is no longer the case in current versions of RSpec

Now about that argument. `context` and `describe` both take an argument. That argument can be a string used to name what you are testing, which RSpec will nicely print to standard output and help show you which tests passed or failed. That is the primary use for those strings, so name them however you want!

```
$ rspec rspec_examples.rb --format documentation

> BankAccount
  can be created with default balance
    is a bank account
    has zero default funds
  with available funds
    can withdraw funds
  with no available funds
    cannot withdraw funds

Finished in 0.00493 seconds (files took 0.09074 seconds to load)
4 examples, 0 failures
```

`describe` and `context` can also take a class name instead of string as an argument. When passing in a class name, RSpec gives you a helper method called `described_class` which is just another way to call on an instance of the class itself.

<h3>IT</h3>

`it` blocks are where your testing usually happens. Just like `describe`, `it` takes an argument of a string that explains the test to the developer. `it` can take a block in which you include the assertions and the set up necessary for that test.

```ruby
it('has zero default funds') do
   is_expected.to(have_attributes({funds: 0 }))
 end
```

Also like `describe`, you can pass a class to `it` but this does not override the top level class you are testing. For example this it block when given class `Integer` will fail, because `described_class` is still `BankAccount`



```ruby
RSpec.describe(BankAccount) do
  describe 'describe_class does not change' do
    it(Integer) do
       expect(described_class).to(be_instance_of(BankAccount))
     end
  end
end
```
```
$ rspec rspec_examples.rb --format documentation

BankAccount
  describe_class does not change
    Integer (FAILED - 1)

Failures:

  1) BankAccount describe_class does not change Integer
     Failure/Error: expect(described_class).to(be_instance_of(BankAccount))
       expected BankAccount to be an instance of BankAccount
     # ./rspec_examples.rb:7:in `block (3 levels) in <top (required)>'

Finished in 0.0157 seconds (files took 0.12 seconds to load)
1 example, 1 failure

Failed examples:

rspec ./rspec_examples.rb:6 # BankAccount describe_class does not change Integer
```

`it` blocks can also be written out as one-liners. You can leave out the argument string describing the test and instead RSpec will print out the assertion itself as the name. The one line syntax relies on either defining a `subject` or passing a class to your example group, which is then implicitly defined as `subject`. So in our main example, there is an implicit definition of an instance of `BankAccount` that is used in the one liners on line 7 and line 10.

<h3>Assertions</h3>

When writing tests, assertions are the methods that are checking the actual results of your code against the expected result. Typically they follow the pattern of `expect(result).to equal(expected_result)`. RSpec is matching the `result` to the `expected_result`. The `equal` is a matcher and RSpec has lots of them, many of them aliases of other ones.

(Table to be formatted)

Matcher    Alias
a_truthy_value    be_truthy
a_nil_value    be_nil
to_not    not_to
an_instance_of    be_a
equal_to    eq


So in line 10, we see that RSpec is checking that the value result of calling `new_bank_account.funds` equals 0.

<h3>Before</h3>

`before` is another RSpec method that we can use to help setup data for our tests. It does what it sounds like and runs before the describe or context blocks. It can be used to set up data that you need to use for all your tests. It takes an argument of `:each` or `:all`.

```
RSpec.describe(BankAccount) do
  before(:each) do
    @bank_account = BankAccount.new
  end

  describe 'can be created with default balance' do
    it('is a bank account') do
       expect(@bank_account).to(be_instance_of(BankAccount))
     end
    it('has zero default funds') do
       expect(@bank_account).to(have_attributes({funds: 0 }))
     end
  end
end
```

`before(:each)` does what it sounds like, it runs before each `it` block.
So in this example the `before` block will run twice.
This means there is no shared data between blocks, as the data is made fresh for each example, but also can slow down your tests if there is unnecessary code being run.


```
RSpec.describe(BankAccount) do
  before(:all) do
    @bank_account = BankAccount.new
  end

  describe 'can be created with default balance' do
    it('is a bank account') do
       expect(@bank_account).to(be_instance_of(BankAccount))
     end
    it('has zero default funds') do
       expect(@bank_account).to(have_attributes({funds: 0 }))
     end
  end
end
```
`before(:all)` will run once at the start and the data will persist between tests, so any changes you make in one test will affect later tests.
If you modify attributes of  `@bank_account` in one `it` statement, they will be remain changed for the next `it` statement

<h3>Let</h3>

The more performant solution would be to use `let`. `let` defines a method whose return value is memoized after the first time it is called. `let` doesn’t run that method until the variable is called for the first time, also known as lazy-evaluation. So the new `BankAccount` from line 19 is not created until it is invoked inside the `it` block on line 21. Similar to before each, the data does not persist. So each new `it` block will start fresh.

To break this block down. `let(:account)` assigns the variable `account`, then the block defines the code that will run the first time the variable is called, in this case creating a new `BankAccount` instance.
Once you call `account` on line 21, the `BankAccount.new` method runs and the result, a BankAccount instance, is assigned to the variable `account`.


`let!` will force the method to run when it is evaluated, bypassing the lazy-loading. This is generally frowned upon due to the performance implications and added confusion to the developer reading the test.

<h2>Takeaways</h2>


- An Rspec spec is built out of `describe` and `context` blocks which have `it` blocks within them.
- `Context` and `Describe` are the aliases for the same method, choosing one of the over is a style preference.
- `it` blocks are where your assertions live, and they are inside of your `describe` and `context` blocks
- `it` blocks are used to group related assertions and set up
- Rspec matchers are used to test the expected outcome to the actual outcome of your code. There are tons of assertions, many of them have aliases.
- The strings passed to `context`, `describe`, and `it` are used to tell the developer what the test is for. These strings are printed to the terminal when running your specs to tell you which tests passed and failed.
- Before hooks run prior to `it` blocks and are used to set up data that is needed for all your tests. Be careful not to create unnecessary data!
- The difference between `let` and `let!` is that `let` is lazy-evaluated, it does not run until it is called the first time where as `let!` runs right away.
