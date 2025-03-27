
---
parent: How To
---
# Optimize DB Queries

### Areas of Focus

- REST APIs
- Bulk operations (e.g. celery tasks)

### Causes of bad performance

- Missing DB indexes (sorting and filtering)
- N+1 queries (these will compound on each other)
 - DB queries in serializer methods (excluding `create()`, `update()`, and validators)

### Patterns

For the below patterns, presume the following models as a starting point:

```python
from django.db import models

class Company(models.Model):
    name = models.CharField()

class Employee(models.Model):
    name = models.CharField()
    retired = models.BooleanField(default=False)
    employer = models.ForeignKey(Company, related_name="employees")
```

#### Additional selections

This strategy correlates to the `QuerySet.select_related()` method. It results in a `JOIN` so it can also be detrimental to performance if it's overused. It is useful to models for which there is a 1-1 relationship.

Example:

```python
# 1 db query
Employee.objects.select_related("employer")
```

#### Prefetching

Prefetching optimizes away N+1 issues by bulk fetching nested data within a query. It is applied by using the `QuerySet.prefetch_related` method.

Basic example:

```python
Company.objects.prefetch_related("employees")
```

A more complex example would be if we want to limit the list of employees to those that aren't retired:

```python
companies = Company.objects.prefetch_related(
    Prefetch("employees", Employee.objects.filter(retired=False))
)
for company in companies:
    for employee in company.active_employees.all():
        print(employee.name)
```

An even better solution is to put this on a different property so we leave `employees` unfiltered:

```python
companies = Company.objects.prefetch_related(
    Prefetch("employees", Employee.objects.filter(retired=False), to_attr="active_employees")
)

for company in companies:
    for employee in company.active_employees.all():
        print(employee.name)
```

Rounding out these examples, since we're now assigning to a separate property, we can update our `Company` model with a custom property to take advantage of this:

```python
class Company:
    ...
    @cached_property
    def active_employees(self):
        # if we've prefetched to _active_employees, use that
        if hasattr(self, "_active_employees"):
            return self._active_employees

        # otherwise fallback to the slower but still accurate N+1 queries
        return self.employees.filter(retired=False)

# both these queries result in the name correctly filtered active_employees list but perform differently
# this ensures we always have correct data but ideally we've prefetched it
# performs N+1 queries
Company.objects.all()

# performs 2 queries
Company.objects.prefetch_related(
    Prefetch("employees", Employee.objects.filter(retired=False), to_attr="_active_employees")
)


```
