# Building Rails 3 custom validators
Every validation extends the base class of ActiveModel::Validator (which handles validating the whole record) or ActiveModel::EachValidator (which handles validating the attribute on the record). The latter module also takes care of :allow_nil and :allow_blank options for you, so the validations are not even invoked when the value doesn’t match the given conditions.
I’m going to give a quick example of how these two base classes differ.

```ruby
# define my validator
class MyValidator < ActiveModel::Validator
  # implement the method where the validation logic must reside
  def validate(record)
    # do my validations on the record and add errors if necessary
    record.errors[:base] << "This is some custom error message"
    record.errors[:first_name] << "This is some complex validation"
  end
end

class Person
  # include my validator and validate the record
  include ActiveModel::MyValidator
  validates_with MyValidator
end

# This method can be used to validate the whole record
# This method below, on the other hand, lets you validate one attribute

class TitleValidator < ActiveModel::EachValidator
  # implement the method called during validation
  def validate_each(record, attribute, value)
    record.errors[attribute] << 'must be Mr. Mrs. or Dr.' unless ['Mr.', 'Mrs.', 'Dr.'].include?(value)
  end
end

class Person
  include ActiveModel::Validations
  # validate the :title attribute with PresenceValidator and TitleValidator
  validates :title, :presence => true, :title => true
end
```

You might have noticed that in the second example, the TitleValidator is “magically” invoked upon the :title => true option in the validates :title statement.
This is another really cool feature that ActiveModel::Validator introduces – all the options passed to validates method stripped of reserved keys (:if, :unless, :on, :allow_blank, :allow_nil) and the remaining keys are resolved to class names with a const_get("#{key.to_s.camelize}Validator") call.
In simple terms it’s a hash key to class name lookup, where everything before *Validator in you class name will be the name of your validation method as an option key.

```ruby
# this is resolved to BarValidator class
validates :name, :bar => true
```

```ruby
# this is resolved to FancyValidationWithPoniesValidator class
validates :name, :fancy_validation_with_ponies => true
```

```ruby
# and this is what is called internally after resolving the class name
validates_with(FancyValidationWithPoniesValidator, options_and_attributes)
```

#Email Validators

```ruby
require 'active_model'
require 'active_support/all'
require 'active_model/validations'
require 'mail'

# http://my.rails-royce.org/2010/07/21/email-validation-in-ruby-on-rails-without-regexp/

class EmailValidator < ActiveModel::EachValidator
  def validate_each(record,attribute,value)
    begin
      return true if value.blank?
      m = Mail::Address.new(value)
      # We must check that value contains a domain and that value is an email address
      r = m.domain && m.address == value
      t = m.__send__(:tree)
      # We need to dig into treetop
      # A valid domain must have dot_atom_text elements size > 1
      # user@localhost is excluded
      # treetop must respond to domain
      # We exclude valid email values like <user@localhost.com>
      # Hence we use m.__send__(tree).domain
      r &&= (t.domain.dot_atom_text.elements.size > 1)
    rescue Exception => e
      r = false
    end
    record.errors[attribute] << (options[:message] || "is invalid") unless r
  end
end
```
#Full Name Validator

```ruby
class FullNameValidator < ActiveModel::Validator
  include NameValidator

  def validate(record)
    [:first_name, :last_name].each do |attribute|
      value = record.send(attribute)
      record.errors[attribute] << "can't be blank" if value.blank?

      if !value.blank?
        record.errors[attribute] << "is too short" if value.size < min_length
        record.errors[attribute] << "is too long" if value.size > max_length
        record.errors[attribute] << "has invalid characters" unless value =~ /^[a-zA-Z\-\ ]*?$/
      end
    end
  end

end

#NameValidator modules which needed in fullname validator

module NameValidator
  mattr_accessor :name_max_length
  mattr_accessor :name_min_length

  def self.name_length range
    @@name_min_length = range.min
    @@name_max_length = range.max
  end

  def min_length
    @@name_min_length || 2
  end

  def max_length
    @@name_max_length || 30
  end
end
```

#Account Number validator

```ruby
class AccountNumberValidator < ActiveModel::EachValidator
  def validate_each(record, attribute, value)
    mod_10(value, record, attribute)
  end

  private

  def mod_10 ccNumb, record, attribute
    # Created by: David Leppek
    #
    # Basically, the alorithum takes each digit, from right to left and muliplies each second
    # digit by two. If the multiple is two-digits long (i.e.: 6 * 2 = 12) the two digits of
    # the multiple are then added together for a new number (1 + 2 = 3). You then add up the
    # string of numbers, both unaltered and new values and get a total sum. This sum is then
    # divided by 10 and the remainder should be zero if it is a valid credit card. Hense the
    # name Mod 10 or Modulus 10.

    valid = "0123456789"  # Valid digits in a credit card number
    len = ccNumb.length   # The length of the submitted cc number
    iCCN = ccNumb.to_i    # integer of ccNumb
    sCCN = ccNumb.to_s    # string of ccNumb
    sCCN = sCCN.strip

    iTotal = 0      # integer total set at zero
    bNum = true     # by default assume it is a number
    bResult = false # by default assume it is NOT a valid cc

    # Determine if the ccNumb is in fact all numbers
    [0..len].each do |j|
      temp = sCCN[j, j + 1]
      bNum = false if !valid.include?(temp)
    end

    # if it is NOT a number, you can either alert to the fact, or just pass a failure
    bResult = false if !bNum

    # Determine if it is the proper length
    if len == 0 && bResult
      # nothing, field is blank AND passed above # check
      bResult = false;
    else
      #ccNumb is a number and the proper length - let's see if it is a valid card number
      if len >= 15
        # 15 or 16 for Amex or V/MC
        [len..0].each do |i|
          # LOOP throught the digits of the card
          calc = iCCN.to_i % 10   # right most digit
          calc = calc.to_i        # assure it is an integer
          iTotal += calc          # running total of the card number as we loop - Do Nothing to first digit
          i--                     # decrement the count - move to the next digit in the card
          iCCN = iCCN / 10        # subtracts right most digit from ccNumb
          calc = iCCN.to_i % 10   # NEXT right most digit
          calc = calc * 2          # multiply the digit by two

          # Instead of some screwy method of converting 16 to a string and then parsing 1 and 6 and then adding them to make 7,
          # I use a simple switch statement to change the value of calc2 to 7 if 16 is the multiple.

          case calc
          when 10
            calc = 1    # 5*2=10 & 1+0 = 1
          when 12
            calc = 3    # 6*2=12 & 1+2 = 3
          when 14
            calc = 5    # 7*2=14 & 1+4 = 5
          when 16
            calc = 7    # 8*2=16 & 1+6 = 7
          when 18
            calc = 9    # 9*2=18 & 1+8 = 9
          else
            calc = calc # 4*2= 8 &   8 = 8  -same for all lower numbers
          end

          iCCN = iCCN / 10  # subtracts right most digit from ccNum
          iTotal += calc  # running total of the card number as we loop
        end

        if (iTotal % 10) ==0
          # check to see if the sum Mod 10 is zero
          bResult = true  # This IS (or could be) a valid credit card number.
        else
          bResult = false # This could NOT be a valid credit card number
        end
      end
    end

    # change alert to on-page display or other indication as needed.
    if !bResult
      record.errors[attribute] << "is not a valid credit card number"
    end
  end
end
```