

Tasks:
1) migrate to the relevant OS (tentative)
2) Switch the Codebase to C++


27th Nov:
Meeting, things to consider:
	About AI Node architecture
	Time taken for model labelling
	If the architecture is set, Can I kick off on model architecture development right away
		or should I work on the communication layer first?
	When should I contact with Tonbo to reduce the 2sec delay


Notes:

54 cameras x 3 streams
24 tata x 1 streams


within 6-9mon:
Finish military set AI node
Finish 2 drone Omnipod

Master AI node comms with Edge AI
Edge AI to store sensor feed too, along with detection
mds_perception -> cpp


Smart connection:
	Feed is only published only when C2 request or if there's a detection -> to reduce bandwidth 


Monday -> networking budget plan smth smth

System architecture finalisation


### Friday (SCRUM) meeting (28/11/2025):

Figure out how tf jira works
	Add stories?

How to explain in sprint meeting:

"My p0 is blahblah, expec to be completed by monday"
"My P1 is balhblah, might take some time, so im planning on doing this and this"
"My P2 bomb developments will be this and that development, mostly done so it might take aroudn 2 days"


P0 - Communication architecture
p1 - Network link budget
P2 - data labelling




Values for each data types
	How much badnwidth needed for each link, per sensor, per ai node, 

Latency analysis:
	Given the maximum distance, between each hops, from sensor-ain
	60KM from aiNode to GCS


From last week action items
Summary:
	Bullet points, layered


Dec3,

JIRA Meetings:

Sprint: one week

EPIC: anything that takes huge amt of time. Example: Tonbo integration, model training
- Larger body, which has sub takes which can be sprints


When to write a story:
- Story shud be part of the EPPIC/Sprint
- Anythign that's collab between dept


Avoid creating story:
- less than 2 day
- Random brainstorming


Title format:
(Action verb) - Component/Area - Outcome

Ex:
- Integrate - EO Camera - Add IR/EO Switch command


In descriptions:
- Purpose of the story: Implement tracking in Tonbo in EO camera
- Deliverables: Code, Docs, Video, Report
- Acceptance criteria: Pan/Tilt must align with 1% error margin


Sub task:
- smaller than story (1 week)
- Sub task is also to be completed within the main task(which is the story, with 1 week timeline)


Updating comments:
- Story should not go to another sprint without a camera
- Anythign changed in the story needs to be commented
- if faced with any blockers



What happens when a story is done on time but the review takes past due date


- No Todo in Sprit -> Todo goes under backlog



- [x] Verify SRT with HM30 and PtoP
- [x] Verify the stream with degraded wifi network, all possible
- Drone is connected to the Omnipod, which receives the stream, it should pump all the received frame to the omnipod, to the mesh network(same mesh the entire Ground unit (Tonbo+comms layer) is present in)
- [ ] Update archi involving Drone's network 
- [x] 