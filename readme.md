Reproducer for issue ISPN-13645

First run the docker compose (at the project root) :

> docker compose up -d 

Then run the kogito process (at the project root) :

> mvn compile quarkus:dev

The kogito process uses the port 8080, make sur it is not already in use.

The kogito process is very simple, it takes an int and create a string the size of the int. It then stop and try to get persisted in infinispan. Using any number above 32765 is going to trigger the bug.

Steps to reproduce the bug :

1. Test the reproducer

> curl -d '{"size": 3}' -H "Content-Type: application/json" -X POST http://localhost:8080/reproducer

Open http://localhost:11222/console/cache/reproducer_domain (infiniadmin:password) to check that the process was correctly persisted.

2. Crash the infinispan :
   
> curl -d '{"size": 35000}' -H "Content-Type: application/json" -X POST http://localhost:8080/reproducer

This time we're creating a string that is above the max size to trigger the "DocValuesField "reproducer.string" is too large, must be <= 32766" error.

Open http://localhost:11222/console/cache/reproducer_domain , the process should still be persisted but the infinispan is now bugged and won't persist anymore.

3. Try again to store a regular object :

> curl -d '{"size": 5}' -H "Content-Type: application/json" -X POST http://localhost:8080/reproducer

Open http://localhost:11222/console/cache/reproducer_domain , see that this time nothing is persisted.
