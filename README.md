# Chatting-Traders

#data science group project

#Hello, this projects attempts to answer the following:


"""How many users are in the database? Deliverable: A number. 
What is the time span of the database? Deliverable: The difference between the largest and the smallest timestamps in the database, a number. 
How many messages of each type have been sent? Deliverable: A pie chart. 
How many discussions of each type have been started? Deliverable: A pie chart. 
How many discussion posts have been posted? Deliverable: A number.
Activity range is the time between the first and the last message (in ANY category) sent by the same user. What is the distribution of activity ranges? Deliverable: a histogram. 
Message activity delay is the time between user account creation and sending the first user message in a specific category. What is the distribution of message activity delays in EACH category? Deliverable: a histogram for each category (ideally all histograms shall be in the same chart, semi-transparent, with legend).
What is the distribution of discussion categories by the number of posts? What is the most popular category? Deliverable: a pie chart, with the most popular category highlighted.
Post activity delay is the time between user account creation and posting the first discussion message. What is the distribution of post activity delays in the most popular category? Deliverable: a histogram. Note: The most popular category shall be carried over from the previous question.
A box plot with whiskers that shows all appropriate statistics for message activity delays in EACH category, post activity delays, and activity ranges.
Finally, you shall write a short report that summarizes your findings in plain English language (for someone who knows neither CS nor Stats). """


import pandas as pd
import datetime

time_Convrs_f = (60 * 60 * 24 * 1000)

from matplotlib import pyplot as plt



""" Step 1.1""" 

Users = pd.read_csv("users.tsv",engine='python' , delimiter='\t+')
Users.columns = ['id', 'memberSince']

print("The number of users is: ")
print(len(Users['id'].unique().tolist()))
print('\n')


"""Step 1.2"""
sortedArray  = Users['memberSince'].min()
sortedArray2 = Users['memberSince'].max()
print("Smallest item is ", sortedArray)
print("Largest item is ",  sortedArray2)

diff = int(sortedArray2-sortedArray)

d = datetime.timedelta(milliseconds=diff)
print("The difference between smallest and largest timestamps in the database is",diff, " miliseconds.")
print ("Or it will be ",d, " in a more human readable format")
print('\n')


"""Step 1.3"""

Messages_org = pd.read_csv("messages.tsv",engine='python',  delimiter='\t+')
Messages_org.columns = ['id', 'sendDate', 'sender_id', 'type'] 


Mess = Messages_org.groupby(['type']).size().reset_index(name='count') 
print('\n')

tx = Mess['count'][0]
ty = Mess['count'][1]

colors = ["#E13F29", "#D69A80"]
plt.pie([tx , ty], labels=['DIRECT_MESSAGE','FRIEND_LINK_REQUEST'], shadow=False,
    # with colors
    colors=colors,
    # with the start angle at 90%
    startangle=90,
    # with the percent listed as a fraction
    autopct='%1.1f%%')

plt.axis('equal')

plt.tight_layout()
plt.title("Messages Type",loc= 'left',pad = 3.0)
plt.savefig('MessagesType.png',dpi= 200)
plt.show()
print('\n')

''' Step 1.4'''
Discussions = pd.read_csv("discussions.tsv",engine='python', delimiter='\t+')
Discussions.columns = ['id', 'createDate', 'creator_id', 'discussionCategory'] 
Discussions_mod = Discussions.groupby(['discussionCategory']).size().reset_index(name = 'category')
print('\n')

#print(df2['category'])
Cate1=Discussions_mod['category'][0] #Eco
Cate2=Discussions_mod['category'][1] #Feed
Cate3=Discussions_mod['category'][2] #Market
Cate4=Discussions_mod['category'][3] #newsreport
Cate5=Discussions_mod['category'][4] #poll
Cate6=Discussions_mod['category'][5] #position
Cate7=Discussions_mod['category'][6] #question
Cate8=Discussions_mod['category'][7] #technical analysis
Cate9=Discussions_mod['category'][8] #technical indicator

plt.pie([Cate1,Cate3,Cate4,Cate5,Cate2,Cate6,Cate7,Cate8,Cate9],
        labels=['ECONOMICEVENT','MARKET_COMMENTARY',
                'NEWSREPORT','POLL','FEED_ITEM','POSITION',
                'QUESTION','TECHNICAL_ANALYSIS',
                'TECHNICAL_INDICATOR'],
                autopct='%1.1f%%' )

plt.axis('equal')
plt.tight_layout()

plt.title("DiscussionsType",loc= 'left',pad = 3.0)
plt.savefig('DiscussionsType.png',dpi= 200)
plt.show()
print('\n')

'''Step 1.5'''

Discussions_posts = pd.read_csv("discussion_posts.tsv",engine='python', delimiter = '\t+')
Discussions_posts.columns = ['id','createDate','discussion_id','creator_id']

print("The number of posts is: ")
print(len(Discussions_posts['id'].unique().tolist()))
print('\n')


''' Step 2 '''

Message_mod = Messages_org.groupby(['sender_id'])
actRange = Message_mod["sendDate"].max() - Message_mod["sendDate"].min()
converted = actRange / time_Convrs_f
plt.title("Activity Range")
plt.hist(converted)
plt.yscale("log")
plt.savefig('ActivityRange.png',dpi= 200)
plt.show()
print('\n')

''' Step 3 '''

Mer = pd.merge(Users,Messages_org,left_on = "id", right_on = "sender_id")
Crct_order = Mer.groupby(["id_x"])
min_ = Crct_order.min()
friend_link = min_[min_['type'] != "DIRECT_MESSAGE"]
friend_link_Delay = (friend_link["sendDate"] - friend_link["memberSince"]) / time_Convrs_f
drcMes = min_[min_['type'] != "FRIEND_LINK_REQUEST"]
drcMes_Delay = (drcMes["sendDate"] - drcMes["memberSince"]) / time_Convrs_f
plt.title("Message Acitivity Delay")
#The Blue is friend link request Orange is Direct message couldn't get the labels to work
plt.xlabel("Blue is Friend Link Request, Orange is Direct Message")
plt.hist(friend_link_Delay, label = ["FRIEND_LINK_REQUEST"])
plt.hist(drcMes_Delay, label = ["DIRECT_MESSAGE"])
plt.yscale("log")
plt.savefig('MessageActivityDelay.png',dpi= 200)
plt.show()
print('\n')

'''Step 4'''

Distr_frame = pd.merge(Discussions_posts, Discussions, left_on="discussion_id",right_on ="id")
DiscussionCount = Distr_frame["discussionCategory"].value_counts()
plt.title("Discussions Posts")
Distr_labels = ["QUESTION", "POLL", "MARKET_COMMENTARY","TECHNICAL_ANALYSIS","POSITION",
             "TECHNICAL_INDICATOR","NEWSREPORT","FEED_ITEM","ECONOMICEVENT"]
plt.pie(DiscussionCount, explode= (0.1,0,0,0,0,0,0,0,0),labels = Distr_labels, autopct ='%1.1f%%')
plt.savefig('DistributionofDiscussion.png',dpi= 200)
plt.show()
print('\n')

''' Step 5 '''

Activity_fram = pd.merge(Discussions_posts ,Users,left_on='creator_id', right_on='id')
ActivOrder = Activity_fram.groupby("creator_id")
Activity_min_ = ActivOrder.min()
time_delay = (Activity_min_["createDate"] - Activity_min_["memberSince"]) / time_Convrs_f
plt.title("Post Activity Delay")
plt.hist(time_delay)
plt.yscale("log")
plt.savefig('PostActivityDelay.png',dpi= 200)
plt.show()
print('\n')

''' Step 6 '''

plt.subplot(1,4,1)
plt.title("Post Activity")
plt.boxplot(time_delay)

plt.subplot(1,4,2)
plt.title("Activity Range")
plt.boxplot(converted)

plt.subplot(1,4,3)
plt.title("Direct Message")
plt.boxplot(drcMes_Delay)

plt.subplot(1,4,4)
plt.title("Friend LRequest")
plt.boxplot(friend_link_Delay)
plt.tight_layout()
plt.savefig('BoxPlot.png',dpi= 200)
plt.show()
