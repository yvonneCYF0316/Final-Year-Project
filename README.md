# Final-Year-Project
SUSTAINABLE INTEGRATION OF RENEWABLE ENERGY SOURCES INTO EXISTING POWER SYSTEMS
#define PWM_PIN 9 // PWM output pin
#define VOLTAGE_PIN 5 // Digital input pin for voltage sensor
#define CURRENT_PIN 6 // Digital input pin for current sensor
#define GRID_PIN 12 
#define REF_VOLTAGE 5.0
#define FULL_BATTERY_VOLTAGE 14.6
#define EMPTY_BATTERY_VOLTAGE 2.5

float solar_voltage; // Voltage reading from sensor (solar)
float solar_power; // Calculated power (solar)
float wind_voltage; // Voltage reading from sensor (wind)
float wind_power; // Calculated power (wind)
float wind_speed; // Speed of wind calculated from voltage
float WindSpeed;
float MPPT_voltage; // Voltage output from MPPT
float voltage_grid; // Voltage from grid
float current_grid;// Current from grid
float Power_Grid; // Calculated power from grid connection
float gridVoltage; // Simulated grid voltage
float outputgrid_Voltage; // Simulated grid output (adapter)
float OUTPUT_GRID_VOLTAGE;
float gridCurrent; //Simulated grid current
float outputgrid_Current; // Simulated grid output current (adapter)
float OUTPUT_GRID_CURRENT;
float power_INPUT;
float power_OUTPUT;
float power_renewable;
float dutyCycle; // Duty cycle for PWM output
float efficiency; // Efficiency for the system
int i=0;

const float voltage_Reference = 5.0; // Assuming 5V reference voltage
const float solar_current = 0.15 ; // current from digital multimeter (solar)
const float wind_current = 0.01 ; // current from digital multimeter (wind)
const float MPPT_current = 1.18 ; // current from dogital multimeter (MPPT)
const float Solar_Voltage_Divider_Ratio = 11.0; // Based on resistor value
const float Wind_Voltage_Divider_Ratio = 10.0; // Based on resistor value
const float maxPower = 15.0; // Maximum power that is able to be handled by the 
system
const float minPower = 2.5; // Minimum power threshold
const float air_density = 1.225 ;
const float swept_area = 0.0047784; 

// Function Prototypes
void readSolar();
void readWind();

// Define constants
const int solar_Voltage_Pin = A0; // Analog pin for solar voltage sensor
const int MPPT_Voltage_Pin = A5; // Analog pin for solar current sensor
const int wind_Voltage_Pin = A2; // Analog pin for wind voltage sensor
const int wind_Current_Pin = A3; // Analog pin for wind current sensor 
const int batteryPin = A4; // Analog pin for battery level
const int relayPin = 8;

// Energy thresholds and setpoints
const int solarThreshold = 20.0; // Threshold value for solar energy activation
const int windThreshold = 5.0 ; // Threshold value for wind energy activation
const int batteryThreshold = 12.0; // Threshold value for battery level

// Control pins
const int solarOutputPin = 2; // Pin for solar output control
const int windOutputPin = 3; // Pin for wind output control
bool gridConnected ; // "false" when grid disconected
void setup() {
 pinMode(solarOutputPin, OUTPUT);
 pinMode(windOutputPin, OUTPUT);
pinMode(PWM_PIN, OUTPUT);
 pinMode(GRID_PIN, INPUT);
 pinMode(relayPin,OUTPUT);
 Serial.begin (9600); 
 randomSeed(analogRead(0));
 digitalWrite(relayPin, LOW);

 // Set up pins
 pinMode(VOLTAGE_PIN,INPUT);
 pinMode(CURRENT_PIN,INPUT);
 
 Serial.begin (9600);
}

void loop() {
 readSolar();
 readWind();
 Serial.print("Solar Voltage Value (V) : ");
 Serial.print(solar_voltage);
 Serial.print(", Solar Current Value (A) : ");
 Serial.println(solar_current);
 Serial.print("Wind Voltage Value (V) : ");
 Serial.print(wind_voltage);
 Serial.print(", Wind Current Value (A) ");
 Serial.print(wind_current);
 Serial.print(", Wind Speed Value (rpm) : ");
 Serial.print(wind_speed);
 Serial.print(", Wind Power : ");
 Serial.print(wind_power);
 Serial.println(" ");
 int batteryLevel = analogRead(batteryPin);
 int gridStatus = analogRead(GRID_PIN);
 solar_power = solar_voltage * solar_current ;
Serial.print ("Solar Power (W): ");
 Serial.print (solar_power);
 
 power_renewable = solar_power + wind_power ;
 Serial.print(", Renewable Energy Power (W): ");
 Serial.println(power_renewable);
 
 int rawMPPT = analogRead(MPPT_Voltage_Pin);
 MPPT_voltage = rawMPPT * (5.0/1023.0) * 4.62;
 Serial.print("MPPT Output Voltage : ");
 Serial.print(MPPT_voltage);
 Serial.print(", MPPT Current : ");
 Serial.println(MPPT_current);
 
 // Solar energy optimization
 if (solar_voltage > 5) {
 digitalWrite(solarOutputPin, HIGH); // Activate solar energy
 digitalWrite(windOutputPin, LOW); // Deactivate wind energy
 gridConnected = false ; // "false" when grid disconected
 }
 
 // Wind energy optimization
 else if (wind_voltage > 1) {
 digitalWrite(solarOutputPin, LOW); // Deactivate solar energy
 digitalWrite(windOutputPin, HIGH); // Activate wind energy
 gridConnected = false ; // "false" when grid disconected; // Disconnect 
from the grid
 }

else 
 gridConnected = true ; // "false" when grid disconected 
 
 {
 //int rawValue=analogRead(batteryPin);
float rawValue = 394.26 ;
 // float batteryVoltage = rawValue * (REF_VOLTAGE/1023.0); //Convert raw value 
to voltage
 float batteryVoltage = 13.2 ;
 
 //Calculate battery percentage
 float batteryPercentage = (batteryVoltage -
EMPTY_BATTERY_VOLTAGE)*100/(FULL_BATTERY_VOLTAGE -
EMPTY_BATTERY_VOLTAGE);
 
 //Maintain percentage between 0 and 100
 batteryPercentage = constrain(batteryPercentage,0,100);
 
 //Display the result on Serial Monitor
 Serial.print("Battery Voltage : ");
 Serial.print(batteryVoltage);
 Serial.print("V, Battery Percentage : ");
 Serial.print(batteryPercentage);
 Serial.println("%");
 
 delay (2000);
 }
 
 // Apply duty cycle to PWM output
 analogWrite(PWM_PIN, dutyCycle);
 
 Serial.print("Duty Cycle: ");
 Serial.print(dutyCycle);
 Serial.println("%");
 
 // Implement MPPT algorithm
 // Adjust duty cycle based on power
if (power_INPUT < minPower) {
 dutyCycle = 100; // Increase the duty cycle to 100% to bring up to MPP
} else if (power_INPUT < (maxPower*0.25)) {
 dutyCycle = 85; // Increase the duty cycle to 85% to bring up to MPP
 } else if (power_INPUT < (maxPower*0.5)) {
 dutyCycle = 70; // Set duty cycle to 70% to regulate to MPP
 }else if (power_INPUT < (maxPower*0.75)) {
 dutyCycle = 60; // Decrease the duty cycle to 60% to bring back to MPP
 } else {
 dutyCycle = 50; // Decrease the duty cycle to 50% to bring back to MPP
 }
 
 // If the grid is connected, turn on relay
 // Otherwise, turn off relay
 //Read grid connection status from analog pin
 if(gridConnected){
 
 digitalWrite(relayPin,LOW);
 gridVoltage = random(228,232); // Simulate grid voltage between 228 V and 232V 
 outputgrid_Voltage = random(188,192); //Output of grid voltage
 OUTPUT_GRID_VOLTAGE = outputgrid_Voltage / 10.0;
 gridCurrent = 32.0; // Value of grid current
 outputgrid_Current = random(236,238); //Output of grid current
 OUTPUT_GRID_CURRENT = outputgrid_Current / 100.0; 
 Serial.println("Grid is Connected");
 } 
 
 else {
 digitalWrite(relayPin,HIGH);
 gridVoltage = 0.0;
 Serial.println("Grid disconnected");
 
 }
//Calculate power
 Power_Grid = OUTPUT_GRID_VOLTAGE * OUTPUT_GRID_CURRENT;
 //Send data to Serial Plotter
 Serial.print("Grid Voltage:");
 Serial.print(gridVoltage);
 Serial.print(", Output Grid Voltage : ");
 Serial.print(OUTPUT_GRID_VOLTAGE);
 Serial.println("V");
 Serial.print("Current:");
 Serial.print(gridCurrent);
 Serial.print(", Output Grid Current : ");
 Serial.print(OUTPUT_GRID_CURRENT);
 Serial.println("A");
 Serial.print("Power:");
 Serial.println(Power_Grid);
 power_INPUT = (MPPT_voltage * MPPT_current)+ power_renewable ; 
 Serial.print ("Input Power : ");
 Serial.print (power_INPUT);
 
 power_OUTPUT = 6.567 ;
 float efficiency = (power_OUTPUT/power_INPUT)*100; 
 Serial.print(", Efficiency : ");
 Serial.print(efficiency);
 Serial.println("%");
 Serial.println(" ");
 // Implement delay or additional control mechanisms as needed
 delay(3000); // Delay between iterations
 }
 
 void readSolar(){
 // Read and convert solar voltage 
 int rawSolarVoltage = analogRead(solar_Voltage_Pin);
 solar_voltage = rawSolarVoltage * (5.0 / 1023.0) * Solar_Voltage_Divider_Ratio;
 }
 
 void readWind() {
 // Read and convert wind voltage
 int rawWindVoltage = analogRead(wind_Voltage_Pin);
 wind_voltage = rawWindVoltage * (5.0 / 1023.0);
 
 // Read and convert wind voltage into wind speed
float WindSpeed = (((wind_voltage - 0.4) / 1.6) * 5.4) * 2.237;
 wind_speed = WindSpeed * 0.44704; // 1 MPH = 0.44704 m/s
 wind_power = 0.5* air_density * swept_area * pow(wind_speed,3); 
}
