---
title: "JPA Specifications"
layout: post
date: 2016-03-18 11:00
tag:
- java
blog: true
---

So, you find yourself having to make changes in a Java application where folks are using [Spring Data](http://projects.spring.io/spring-data-jpa/) based [repositories](http://www.martinfowler.com/eaaCatalog/repository.html). Everything looks good in the beginning, no need to implement any [ORM](https://en.wikipedia.org/wiki/Object-relational_mapping) code from scratch since Spring Data will figure that out for us. Let’s see an example.

{% highlight java %}
@Entity
public class Customer {
  @Id
  private Integer id;
  private String firstName;
  private String lastName;
  private LocalDate birthDate;
  private LocalDate enrollment;
  @Enumerated(STRING)
  private FinancialStatus status;

  // more stuff
}

public enum FinancialStatus {
  GOOD_STANDING,
  DEBT,
  BANKRUPT,
  DECEASED
}
{% endhighlight %}

Given the above `Customer` entity, imagine you have the following repository interface:

{% highlight java %}
public interface CustomerRepository extends CrudRepository<Customer, Integer> {

  List<Customer> findByLastNameAndStatusIn(String lastName, List<FinancialStatus> statuses);

  List<Customer> findByFirstNameAndLastNameOrderByEnrollmentDesc(String firstName, String lastName);
}
{% endhighlight %}

As you may have realized, just by extending `CrudRepository` you get all the basic stuff like `save`, `findAll`, `findOne` for free. Then, anything more complex, like the two declared methods, will be interpreted by Spring Data in runtime based on name conventions.

## The Problem

As the time goes on and business start to ask more complicated questions, your repository starts to get weird. Enhancing our previous example to illustrate that results in the following repository code:

{% highlight java %}
public interface CustomerRepository extends CrudRepository<Customer, Integer> {

  List<Customer> findByLastNameAndStatusInOrderByEnrollmentDesc(String lastName, List<FinancialStatus> statuses);

  List<Customer> findByFirstNameAndLastNameAndBirthDateIsLessThanAndStatusInOrderByEnrollmentDesc(String firstName, String lastName, LocalDate birthDateLimit, List<FinancialStatus> statuses);

  List<Customer> findByFirstNameAndLastNameAndBirthDateIsBetweenAndStatusInOrderByEnrollmentDesc(String firstName, String lastName, LocalDate birthDateFrom, LocalDate birthDateTo, List<FinancialStatus> statuses);
 }
{% endhighlight %}

At this point, the maintenance of this particular repository starts to become problematic for obvious reasons:

- Method names are too long and excessively verbose;
- They’re not easy to read;
- It’s not straightforward to build new queries (as we need to rely on Spring Data ability to understand the method name);
- Some method names don’t even result in a valid English sentence.

Imagine yourself talking to your pair:

> Hey [Akon](https://twitter.com/konwi7), I think we could use the **find by first name and last name and birth date is between and status in order by enrollment desc** method here.

Awkward, to say the least.

Another problem that may call your attention is the duplication of criteria among the query methods: most of them are repeating the same filters over and over again. And as the amount of filters increase, the argument list also increases making it more difficult to figure out what is what.

Wouldn’t that be great if we could extract those filters in smaller reusable pieces?

## JPA Specifications

With the Spring Data [JPA Specifications](http://bit.ly/1KV77Ee) API we can implement this idea of extracting all filter and sorting criteria in independent pieces. Let’s see the new version of the the repository looks like:

{% highlight java %}
public interface CustomerRepository extends CrudRepository<Customer, Integer>, JpaSpecificationExecutor<Customer> {
  class Specifications {
    public static Specification<Customer> firstNameIs(String firstName) {
      return (root, query, cb) -> cb.like(root.get("firstName"), firstName);
    }

    public static Specification<Customer> lastNameIs(String lastName) {
      return (root, query, cb) -> cb.like(root.get("lastName"), lastName);
    }

    public static Specification<Customer> statusIn(FinancialStatus... statuses) {
      return (root, query, cb) -> cb.isTrue(root.get("status").in(statuses));
    }

    public static Specification<Customer> birthDateIsBetween(LocalDate from, LocalDate to) {
      return (root, query, cb) -> cb.between(root.get("birthDate"), from, to);
    }

    public static Specification<Customer> birthDateIsBefore(LocalDate limit) {
      return (root, query, cb) -> cb.lessThan(root.get("birthDate"), limit);
    }

    public Sort orderByEnrollmentDesc() {
      return new Sort(Sort.Direction.DESC, "enrollment");
    }
  }
{% endhighlight %}

As you can see in the new version of the repository, we had to extend `JpaSpecificationExecutor`. That makes it possible for our repository to accept `Specification` based queries using the following method:

{% highlight java %}
List<T> findAll(Specification<T> spec, Sort sort);
{% endhighlight %}

After that,  we've implemented each of the necessary filters and sorting as static methods and got rid of all the old long-named methods. The inner `Specifications` class works as a namespace just to avoid Spring Data trying to interpret our filters as regular `CrudRepository` methods.

Without getting too deep in details about `root`, `query`, `cb` (criteria builder), what our new methods do is to return reusable specifications. And that should be enough to allow us to replace calls for

`findByFirstNameAndLastNameAndBirthDateIsLessThanAndStatusInOrderByEnrollmentDesc`

with the following code:

{% highlight java %}
List<Customer> results = repository.findAll(
   where(firstNameIs("John"))
     .and(lastNameIs("Doe"))
     .and(birthDateIsBefore(eighteenYearsAgo))
     .and(statusIn(GOOD_STANDING, DEBT)),
   orderByEnrollmentDesc());
{% endhighlight %}

The example above, alone, would be enough to make most people happy when it comes to readability and flexibility. Still, we could even go further and define default methods with meaningful names in our new repository using our new specifications like this:

{% highlight java %}
  default List<Customer> inDebtWithLastName(String lastName) {
    return findAll(where(statusIn(DEBT)).and(lastNameIs(lastName)));
  }
{% endhighlight %}

## Conclusion

The magic Spring Data does, allowing us to define queries only by writing method names is definitely cool for most of the simpler situations. However, it doesn’t scale well as queries become more complex. The Specification API, as described, seems to be a good alternative. Keep that in your tool-belt.
