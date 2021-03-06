[![Build Status](https://travis-ci.com/mtumilowicz/hibernate-batch-size.svg?branch=master)](https://travis-ci.com/mtumilowicz/hibernate-batch-size)

# hibernate-batch-size
The main goal of this project is to provide simple solution to 
N+1 query problem.

# preface
A naive `O/R` implementation would `SELECT` entity and then do 
additional SELECTs for getting the information of child entities,
so in case of (ex. `@OnToMany`) mapped collection of child entities 
(`size = N`) causes **N+1 query problem**.

# solution
**`@BatchSize`** - Defines size for batch loading of collections or 
lazy entities.

Without `@BatchSize`, you’d run into a N+1 query issue.

However, although `@BatchSize` is better than running into an `N+1` query 
issue, most of the time, a `DTO` projection or a `JOIN FETCH` is a much 
better alternative since it allows you to fetch all the required data 
with a single query.

# example
* over entity
    ```
    @Entity
    @BatchSize(size=100)
    class Product {
        ...
    }
    ```
    will initialize up to 100 lazy Product entity proxies at a time.

* over relation
    ```
    @OneToMany
    @BatchSize(size = 5) /
    Set getProducts() { ... };
    ```
    will initialize up to 5 lazy collections of products at a time

# tests
* Running `withBatch()` tests produces sql queries:
    ```
    Hibernate: select employee0_.id as id1_0_0_ from employee employee0_ where employee0_.id=?
    Hibernate: select employee0_.id as id1_0_0_ from employee employee0_ where employee0_.id=?
    Hibernate: select employee0_.id as id1_0_0_ from employee employee0_ where employee0_.id=?
    
    Hibernate: select batchissue0_.batch_issues_id as batch_is3_1_1_, batchissue0_.id as id1_1_1_, batchissue0_.id as id1_1_0_ from issue batchissue0_ where batchissue0_.batch_issues_id in (?, ?, ?)
    ```
    
* Running `withoutBatch()` tests produces sql queries:
    ```
    Hibernate: select employee0_.id as id1_0_0_ from employee employee0_ where employee0_.id=?
    Hibernate: select employee0_.id as id1_0_0_ from employee employee0_ where employee0_.id=?
    Hibernate: select employee0_.id as id1_0_0_ from employee employee0_ where employee0_.id=?
    
    Hibernate: select notbatchis0_.not_batch_issues_id as not_batc2_1_0_, notbatchis0_.id as id1_1_0_, notbatchis0_.id as id1_1_1_ from issue notbatchis0_ where notbatchis0_.not_batch_issues_id=?
    Hibernate: select notbatchis0_.not_batch_issues_id as not_batc2_1_0_, notbatchis0_.id as id1_1_0_, notbatchis0_.id as id1_1_1_ from issue notbatchis0_ where notbatchis0_.not_batch_issues_id=?
    Hibernate: select notbatchis0_.not_batch_issues_id as not_batc2_1_0_, notbatchis0_.id as id1_1_0_, notbatchis0_.id as id1_1_1_ from issue notbatchis0_ where notbatchis0_.not_batch_issues_id=?
    ```