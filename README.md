# Webots
#include <math.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <webots/camera.h>
#include <webots/motor.h>
#include <webots/range_finder.h>
#include <webots/robot.h>
#include <webots/receiver.h>
#include <webots/distance_sensor.h>


#define TIME_STEP 64
#define MAX_SPEED 5.24
#define CRUISING_SPEED 5
#define TOLERANCE -0.1
#define OBSTACLE_THRESHOLD 0.5
#define SLOWDOWN_FACTOR 0.5
#define COMMUNICATION_CHANNEL 1

static WbDeviceTag camera;
static WbDeviceTag left_wheel, right_wheel;
static WbDeviceTag kinectColor;
static WbDeviceTag kinectRange;  
static WbDeviceTag communication;

const float *kinect_values;
static int time_step = 0;
static int tps =32;
//Any variables that you may needs to access at 
int r=0;
int g=0;
int b=0;



double red = 100;
double blue =100; 
int message_printed = 0; /* used to avoid printing continuously the communication state */

static void initialize(){
  // necessary to initialize Webots
  wb_robot_init();
  // get time step and robot's devices
  time_step = wb_robot_get_basic_time_step();
  communication = wb_robot_get_device("receiver");
  wb_receiver_enable(communication, time_step);
  //Add any sensors you need to delcare here add any other sensors declare them under here look at the base pioneer3dx code to try to work out which variables need to be declared
  time_step = wb_robot_get_basic_time_step();
  left_wheel = wb_robot_get_device("left wheel");
  right_wheel = wb_robot_get_device("right wheel");
  kinectColor = wb_robot_get_device("kinect color");
  kinectRange = wb_robot_get_device("kinect range");
  
  
  
  wb_camera_enable(kinectColor, time_step);
  wb_range_finder_enable(kinectRange, time_step);
  wb_motor_set_position(left_wheel, INFINITY);
  wb_motor_set_position(right_wheel, INFINITY);
  wb_motor_set_velocity(left_wheel, 0.0);
  wb_motor_set_velocity(right_wheel, 0.0);
  WbDeviceTag camera = wb_robot_get_device("camera");
  wb_camera_enable(camera, time_step);
}
  
  
static int homeostasis(){
//You must not change this function. This is a simple decay of the Blue and Red resouces. you may only comment out the print statement
 red-=(1.0/tps);
 blue-=(1.0/tps);
 printf ("My red is %f  and my blue is %f\n", red, blue);
 if (red <=0 || blue <0){
   return false;}
  else {
    return true;}
}

static void recoverRed(){
//You must not change this function
  if (red <=100){
  red +=1;
}}

static void recoverBlue(){
//You must not change this function
  if (blue <=100){
 blue+=1;
}}  

static void message(){
//You may change this function but be very careful!
  int message_printed = 0; 
  if (wb_receiver_get_queue_length(communication) > 0) {
    const char *buffer = wb_receiver_get_data(communication);
    if (message_printed != 1) {
          message_printed = 1;
        }
    int redCheck =1;
    int blueCheck =1;
    redCheck = strcmp(buffer,"Red");
    blueCheck = strcmp(buffer, "Blue");
    if (redCheck == 0){
       recoverRed();
       }
    if (blueCheck == 0){
       recoverBlue();
      }
     wb_receiver_next_packet(communication);            
 }}

// Add any new functions you need under this I've give a few dummies to get you started


static void move(int l,int r){
 int left_speed = l;
 int right_speed =r;
 wb_motor_set_velocity(left_wheel, 5);
 wb_motor_set_velocity(right_wheel, 5);
}  

static void vision(){
  r=0;
  g=0;
  b=0;
  camera = wb_robot_get_device("camera");
  const unsigned char *image= wb_camera_get_image(camera);
  int image_width = 64;
  int image_height = 64; 
  for (int x = 0; x < image_width; x++){
    for (int y = 0; y < image_height; y++) {
      r += wb_camera_image_get_red(image, image_width, x,y);
      g += wb_camera_image_get_green(image, image_width, x,y);
      b += wb_camera_image_get_blue(image, image_width, x,y);}}
  r =r/4096;
  g =g/4096;
  b =b/4096; 
  if (r > 240 && g && b < 20){
    move(5,5);  
  }   
}

static void avoidObstacle(){
  WbDeviceTag s0 = wb_robot_get_device("so0");
  wb_distance_sensor_enable(s0, time_step);
  float z = wb_distance_sensor_get_value(s0);
  printf("%f \n", wb_distance_sensor_get_value(s0));
    if (z > 600){
      wb_motor_set_velocity(left_wheel,2);
      wb_motor_set_velocity(right_wheel,5);   
    }
}

//static void detectObstacles(){}

//static void explore(){}


int main() {  
    initialize();
  while (wb_robot_step(time_step) != -1 && homeostasis() == true) { // While the robot is alive
  move(5,5);
  message();
  vision();
  printf("red= %d, green= %d, blue= %d \n", r,g,b);
  avoidObstacle();
  }
  move(0,0);
  printf ("Shutting down");
  wb_robot_cleanup(); 
  }
