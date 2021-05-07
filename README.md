using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

using Amazon.Lambda.Core;

using Newtonsoft.Json;
using Amazon.DynamoDBv2;
using Amazon.DynamoDBv2.DataModel;
using Amazon.Lambda.APIGatewayEvents;
using Amazon.DynamoDBv2.Model;
using Amazon.DynamoDBv2.DocumentModel;

// Assembly attribute to enable the Lambda function's JSON input to be converted into a .NET class.
[assembly: LambdaSerializer(typeof(Amazon.Lambda.Serialization.Json.JsonSerializer))]

namespace govSpendingQuery
{

    public class stateEntry
    {
        public string fips;
        public string name;
        public string code;
        public string count;
        public string type;
        public double amount;
    }

    public class Function
    {
        private static AmazonDynamoDBClient client = new AmazonDynamoDBClient();
        private string tableName = "StateSpending";

        public async Task<stateEntry> FunctionHandler(APIGatewayProxyRequest input, ILambdaContext context) 
        { 

            //-Create variable to hold incoming object represented as a  JSON string
            //--------------------------------------------------------
            string stateAbbrev = "";

            //-Create a dictionary to hold incoming query string params
            //---------------------------------------------------------
            Dictionary<string, string> dict = (Dictionary<string, string>)input.QueryStringParameters;

            //- search the query string for the value 'state'.  When found, output contents to variable stateAbbrev
            //----------------------------------------------------------
            dict.TryGetValue("state", out stateAbbrev);

            //query the Dynamo DB Table.  Put the results into a dictionary.  Assign the dictionary Key 'name' and the values
            //is the rest of the database info
            //----------------------------------------------------------
            GetItemResponse res = await client.GetItemAsync(tableName, new Dictionary<string, AttributeValue>
            {
                { "code", new AttributeValue {S = stateAbbrev } }
            }
            );

            //- Convert data from the table into a document
            Document myDoc = Document.FromAttributeMap(res.Item);

            //- convert document into an object
            stateEntry myItem = JsonConvert.DeserializeObject<stateEntry>(myDoc.ToJson());

            return myItem;
        }
    }
}
