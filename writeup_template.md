## Project: 3D Motion Planning
![Quad Image](./misc/enroute.png)

---


# Required Steps for a Passing Submission:
1. Load the 2.5D map in the colliders.csv file describing the environment.
2. Discretize the environment into a grid or graph representation.
3. Define the start and goal locations.
4. Perform a search using A* or other search algorithm.
5. Use a collinearity test or ray tracing method (like Bresenham) to remove unnecessary waypoints.
6. Return waypoints in local ECEF coordinates (format for `self.all_waypoints` is [N, E, altitude, heading], where the drone’s start location corresponds to [0, 0, 0, 0].
7. Write it up.
8. Congratulations!  Your Done!

## [Rubric](https://review.udacity.com/#!/rubrics/1534/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it! Below I describe how I addressed each rubric point and where in my code each point is handled.

### Explain the Starter Code

#### 1. Explain the functionality of what's provided in `motion_planning.py` and `planning_utils.py`

`Motion Planning:` defines states and callbacks so that our drone can operate completely (taking off, landing and moving). This code has also an uninplemented section dedicated to path plannig, which reads the obstacles that the drone should face and performs a search to reach the destination by avoiding theses obstacles while optimizing the number of waypoints.

`Planning Utils:` defines planning-related functions such as a creator for grid representation, a verifier for valid actions and a a_star implementations, with the ecessary heuristic function. 
And here's a lovely image of my results (ok this image has nothing to do with it, but it's a nice example of how to include images in your writeup!)

### Implementing Your Path Planning Algorithm

#### 1. Set your global home position
This was done by simply reading the first two lines of the csv and converting the values to float, as shown below:

       with open('colliders.csv', newline='') as colliders:
            reader = csv.reader(colliders)
            origin = next(reader)
            lat0 = np.float64(origin[0].split(' ')[1].lstrip())
            lon0 = np.float64(origin[1].split(' ')[2].lstrip())
            

#### 2. Set your current local position
To set my current local position, I´ve used `global_to_local` function, as can be seen below.

        self.set_home_position(lon0, lat0, 0)

        # TODO: retrieve current global position
        global_position = self.global_position

        # TODO: convert to current local position using global_to_local()
        current_local_position = global_to_local(global_position, self.global_home)


#### 3. Set grid start position from local position

`current_position = (int(self.local_position[0]+grid_start[0]), int(self.local_position[1]+grid_start[1]))`

#### 4. Set grid goal position from geodetic coords
I´ve defined the goal position and used `global_to_local` function, accounting for the offsets and forcing to be an integer.

        goal_local_pos = global_to_local([-122.401950, 37.794433, self.global_home[2]], self.global_home)
        grid_goal = (int(goal_local_pos[0] - north_offset), int(goal_local_pos[1] - east_offset))
        
#### 5. Modify A* to include diagonal motion (or replace A* altogether)
I´ve performed this task in two parts, the first one being to add the action definitions for diagonal motion as shown:

    NORTH = (-1, 0, 1)
    NORTH_EAST = (-1, 1, sqrt(2))
    EAST = (0, 1, 1)
    SOUTH_EAST = (1, 1, sqrt(2))
    SOUTH = (1, 0, 1)
    SOUTH_WEST = (1, -1, sqrt(2))
    NORTH_WEST = (-1, -1, sqrt(2))
    WEST = (0, -1, 1)

After the actions were created, I´ve added the necessary validity check:

    if x - 1 < 0 or grid[x - 1, y] == 1:
        valid_actions.remove(Action.NORTH)
    if x - 1 < 0 or y + 1 > m or grid[x - 1, y + 1] == 1:
        valid_actions.remove((Action.NORTH_EAST))
    if y + 1 > m or grid[x, y + 1] == 1:
        valid_actions.remove(Action.EAST)
    if x + 1 > n or y + 1 >m or  grid[x+1, y+1] == 1:
        valid_actions.remove(Action.SOUTH_EAST)
    if x + 1 > n or grid[x + 1, y] == 1:
        valid_actions.remove(Action.SOUTH)
    if x + 1 < 0 or y - 1 < 0 or grid[x+1, y-1] == 1:
        valid_actions.remove(Action.SOUTH_WEST)
    if y - 1 < 0 or grid[x, y - 1] == 1:
        valid_actions.remove(Action.WEST)
    if x - 1 < 0 or y - 1 < 0 or grid[x-1, y-1] == 1:
        valid_actions.remove(Action.NORTH_WEST)

#### 6. Cull waypoints 
Here I´ve worked with a simple yet effective path pruning, checking for changes in each dimension. The downside of this approach is the lack of pruning for diagonal moves:

            n = 0
            i = 0
            new_path = [path[n+i]]
            # let's check if we have steps in which we change only one coordinate
            while n + 1 < len(path):
                # we check for similarities on each step, stopping at a difference (or at a safety distance)
                while n + i + 1 < len(path):
                    if path[n][0] == path[n + i][0]  and i < 7:
                        i += 1
                    else:
                        break
                while n + i + 1 < len(path):
                    if path[n][1] == path[n + i][1]  and i < 7:
                        i += 1
                    else:
                        break
                n += i
                new_path.append(path[n])
                i = 0

        return new_path 


### Execute the flight
#### 1. Does it work?
It works!

### Double check that you've met specifications for each of the [rubric](https://review.udacity.com/#!/rubrics/1534/view) points.
  
# Extra Challenges: Real World Planning

For an extra challenge, consider implementing some of the techniques described in the "Real World Planning" lesson. You could try implementing a vehicle model to take dynamic constraints into account, or implement a replanning method to invoke if you get off course or encounter unexpected obstacles.


