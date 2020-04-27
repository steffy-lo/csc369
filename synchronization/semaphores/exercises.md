# Exercises

![](../../.gitbook/assets/image%20%2831%29.png)

### Dining Philosopher's Problem

There are 5 philosophers seated at a table with one chopstick between each. A philosopher needs two chopsticks to eat. 

![Dining Philosopher&apos;s Problem](../../.gitbook/assets/image%20%2816%29.png)

1. Mutual Exclusion
   * Only one philosopher can hold a chopstick at the same time
2. Hold and Wait
   * A philosopher will pick up the first chopstick then wait until they can pick up the next chopstick
3. No preemption
   * A philosopher cannot grab a chopstick from their neighbour
4. Circular Wait
   * If each philosopher picks up their left chopstick at the same time, then they will all be waiting to pick up their right chopstick \(i.e., deadlock\)
   * To prevent this deadlock, the simplest solution is to make one philosopher pick up their right chopstick first. This will prevent circular wait

