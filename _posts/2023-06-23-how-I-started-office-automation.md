# How I started office automation

#### Category: Diaries & Journals

---

### Throwback – My first job

### Pressure and motivation

My first actual job in the office was a temporary assignment. Having no finite period of the contract, the company kept extending the term instead of hiring me as a permanent employee. I thought it was just a blood-draining experience that I had to prove that I am the efficient and smart one who can add more value than the current employees. Hoping that I replace someone else sounds cruel but that was the reality unless they create a new spot. I even had to believe the sugar-coated promise the director made that she would create a position for me someday. Sorry for the gloomy story for an open-up. Although I was under pressure, I believe that was the period that I gained the most skills during a short period and I somehow performed better under the right amount of pressure. Maybe I was just eager enough to endure the pressure at the time.

### A good friend can influence your success.

One of my closest friends and I had the most boring Saturday routine that guys in their mid-20s can imagine. We met at one of the Starbucks in the town and did our own stuff until sunset. When the sun goes down and we feel starved, we went to a pub talking about random bs, mostly politics, money, career, etc. What was our own stuff done with coffee? I studied for courses that I was taking to enter the CPA program and he worked. Yes, he worked on Saturday. Although looking back, we both agreed that there was no work-life balance that he was swamped by reports that needed to be done during the weekend and I once had the same path, I think I admired it at the time because, before the pandemic, it was not common for companies to give a laptop to their employees unless they are in law, audit, or mid-senior level in the industry.

He’s been a good mentor in my career and helped me in many ways including reviewing my resume and coaching interview. He was already on a good track in his career, at least that’s what I thought, so I set him as a benchmark. It looked even cool to me that he does the work on the spreadsheet using shortcut keys. So, I had an [Excel blog](https://jaykim361.tistory.com/category/%EC%97%91%EC%85%80%202016) always opened on my iPhone and read it whenever I had downtime at work. I’ve drilled down each post and tried to use them at work. I still believe that was one of the best things I’ve chosen to do as I am now the Excel guy at work that people look for when they are stuck with the sheet. People may wonder why Excel so much matters to me but I believe the small differences in your skillset make you distinguished from many candidates with similar experience. It’s not just Excel. When I became equipped with my own unique combination of skills and experience, I noticed that I had relatively fewer competitors when the combination is in demand. Excel was just the beginning of all.

### If necessity is the mother of invention, laziness is probably the father.

Back to the story of my first job, I noticed that there were some processes that looked obviously inefficient to me for whatever reason such as resistance to change. Staff were given tedious tasks as if the plant manager tries to purposefully maintain her overall equipment effectiveness (OEE) high in a manufacturing facility or plantation. Among many, I always was not really a fan of any type of data entry regardless of its volume. We created an Excel (or CSV) file for the system setup data or employee data based on the collective agreements between the company and unions. Then they needed to be entered into the payroll management system. Not only it is obviously slower but I always believed that it is more prone to human errors such as typos when it was done manually. It only makes sense for me to invest more time when there is a benefit such as increased accuracy. So, investing more staff hours for less accurate results looked absolutely impractical to me.

Having no prior experience in writing any script, it was tough but the learning curve was stiff. Every line of it was challenging but the Google was my prof. Most questions that I had for subsets of the procedures were already asked and answered in Stack Overflow but I sometimes had to invent my own solution. It seemed that controlling the AS400-based payroll system via Excel VBA was impossible, at least on my level at the time. So, I had to come up with a walkaround and decided to break up the process into two. When I noticed that the AS400 saves its macros in .txt format (Now I know it is in .vbs format to be more accurate), I decided to create the macro script in the text file that the system could read. So, exporting the setup or employee data from the CSV files into the text format that could be read by the payroll system solved the issue. Well, there was no free lunch. It was usual that I had to stay at the company till 8 or 9 in an empty office but I considered that an investment.

![entering-data-from-csv-file-to-as400](/images/entering-data-from-csv-file-to-as400.gif)
Entering data from the CSV file to AS400

### Gaining more responsibilities

Now I have hours saved. As I kept proving that I could do better than what they initially hired me for, I started to gain more responsibility. Once I proved my ability to analyze the legal contracts, I was delegated to craft the structure for the system parameters based on the collective agreements. Once they noticed that I have great Excel skills, they started to approve the initiatives I suggested for improving processes and templates. Among many goals I wanted to achieve, my primary interest was in improving accuracy and saving time. To enhance accuracy, drop-down menus were implemented in all templates so that users are not allowed any manual entry. Also, the manual calculation was eliminated and the calculated fields were locked. Data validation for the reasonability checks for the inputs such as the range of numerical values and length of letters were added to the forms. All these helped the head corporate office stop receiving the form submitted with inaccurate or incomplete data that used to require it to reach out to the submitters back to ask for a resubmission or correct information.

The other project that I really enjoyed was streamlining the submission process of forms where I automated the printing of them so the physical copies can be stored in branch managers’ cabinets, saving the file in the network folder for the corporate office’s record-keeping purposes, and sending an email with the attachment to the corporate processing team’s shared email so the request sits in the inbox until it gets processed. All of them were designed to be done right with a single click on the submit button to prevent misplacement of the files and enable tracking of whether the request was processed or not. The whole point was to maintain the integrity of the process by eliminating the processes instead of expecting the branch managers to follow SOPs.

![print-save-and-send](/images/print-save-and-send.gif)
Print, save and send

### Lending a hand to others

Not all coworkers were not happy with the changes that I brought to the team. It was even way before people started talking about ‘AI’ and most people thought automating tasks would make human labour obsolete. My first offer of help with sharing the macro template of data entry was actually turned down by the data entry clerk when she was scared of losing her job and would not be able to find the other one. I found my skills and mindset were mostly welcomed by the younger supervisors who would be more likely to take risks to climb up the corporate ladder. When I walked by someone and saw something I can make their life easier, I didn’t hesitate to offer help but it was absolutely up to them to take it or not. One example was to split a big batch file into 51 smaller ones when each 51 branch managers was not supposed to see the data belonging to the other branches. When the process of ‘filtering data -> saving as a new file with the filtered data -> unfiltering the data’ was repeated 51 times on a bi-weekly basis, it seemed a pretty boring task and it was a no-brainer offer.

![filter-save-the-subset-and-send](/images/filter-save-the-subset-and-send.gif)
Filter, save the subset and send

### Final words

Anyone can automate tasks. The room for improvement all around the world. It may eliminate some jobs but will create other new ones. Let’s keep automating, saving time, and spending saved time creating values in more creative ways.

![anyone-can-cook](/images/anyone-can-cook.png)
