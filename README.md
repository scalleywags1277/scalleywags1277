        public string getAuthFullFlow(string username, string password)
        {
            var cli1 = new RestClient("https://www.epicgames.com/id/api/csrf");
            var req1 = new RestRequest(Method.GET);
            var ans1 = cli1.Execute(req1);
            var xsrftoken = ans1.Cookies.First(x => x.Name == "XSRF-TOKEN").Value;

            var cli2 = new RestClient("https://www.epicgames.com/id/api/login");
            var req2 = new RestRequest(Method.POST);
            foreach (var i in ans1.Cookies)
            {
                req2.AddCookie(i.Name, i.Value);
            }

            req2.AddHeader("x-xsrf-token", xsrftoken);
            req2.AddHeader("Content-Type", "application/x-www-form-urlencoded");
            req2.AddParameter("email", username);
            req2.AddParameter("password", password);
            req2.AddParameter("rememberMe", true);
            var ans2 = cli2.Execute(req2);

            var cli3 = new RestClient("https://www.epicgames.com/id/api/redirect");
            var req3 = new RestRequest(Method.GET);
            req3.AddHeader("x-xsrf-token", xsrftoken);
            req3.AddHeader("Referer", "https://www.epicgames.com/id/login");
            foreach (var i in ans2.Cookies)
            {
                req3.AddCookie(i.Name, i.Value);
            }
            var ans3 = cli3.Execute(req3);

            var cli4 = new RestClient("https://www.epicgames.com/id/api/authenticate");
            var req4 = new RestRequest(Method.GET);
            foreach (var i in ans3.Cookies)
            {
                req4.AddCookie(i.Name, i.Value);
            }
            var ans4 = cli4.Execute(req4);

            var cli5 = new RestClient("https://www.epicgames.com/id/api/exchange");
            var req5 = new RestRequest(Method.GET);
            req5.AddHeader("x-xsrf-token", xsrftoken);
            foreach (var i in ans4.Cookies)
            {
                req5.AddCookie(i.Name, i.Value);
            }
            var ans5 = cli5.Execute(req5);

            var cli6 = new RestClient("https://account-public-service-prod03.ol.epicgames.com/account/api/oauth/token");
            var req6 = new RestRequest(Method.POST);
            req6.AddHeader("Content-Type", "application/x-www-form-urlencoded");
            req6.AddHeader("Authorization", "basic MzQ0NmNkNzI2OTRjNGE0NDg1ZDgxYjc3YWRiYjIxNDE6OTIwOWQ0YTVlMjVhNDU3ZmI5YjA3NDg5ZDMxM2I0MWE=");
            req6.AddParameter("grant_type", "exchange_code");
            req6.AddParameter("exchange_code", JsonConvert.DeserializeObject<CodeClass>(ans5.Content).code);
            req6.AddParameter("includePerms", true);
            req6.AddParameter("token_type", "eg1");
            var ans6 = cli6.Execute(req6);
            var token = JsonConvert.DeserializeObject<Ans6ParseClass>(ans6.Content).access_token;
            return token;
        }
        //Crappy botch cause im bad with json.net
        public class CodeClass
        {
            public string code { get; set; }
        }
        public class Ans6ParseClass
        {
            public string access_token { get; set; }
        }
