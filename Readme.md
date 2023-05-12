using Microsoft.Web.WebView2.Core;
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace InspectElementTool
{
    public partial class Form1 : Form
    {
        private CoreWebView2 webView2;

        public Form1()
        {
            InitializeComponent();
        }

        private async void Form1_Load(object sender, EventArgs e)
        {
            // Initialize the WebView2 control
            await webView2.EnsureCoreWebView2Async();

            // Navigate to a web page
            webView2.CoreWebView2.Navigate("https://www.example.com/");
        }

        private async void webView2_CoreWebView2Ready(object sender, EventArgs e)
        {
            // Wait for the WebView2 control to finish initializing
            webView2 = (CoreWebView2)sender;

            // Wait for the web page to finish loading
            await webView2.CoreWebView2.DOMContentLoadedAsync();

            // Retrieve the DOM tree of the web page
            string script = "JSON.stringify(document.documentElement.outerHTML)";
            string domTreeJson = await webView2.CoreWebView2.ExecuteScriptAsync(script);

            // Parse the DOM tree and populate the ListBox control
            var doc = new HtmlAgilityPack.HtmlDocument();
            doc.LoadHtml(domTreeJson);
            PopulateTreeView(doc.DocumentNode.ChildNodes, treeView1.Nodes);

            // Add an event handler for the ListBox control's SelectedIndexChanged event
            treeView1.AfterSelect += async (s, args) =>
            {
                // Retrieve the HTML, CSS, and JavaScript code of the selected element
                string script2 = "JSON.stringify(arguments[0].outerHTML);" +
                    "JSON.stringify(arguments[0].style.cssText);" +
                    "JSON.stringify(arguments[0].attributes)";
                string result = await webView2.CoreWebView2.ExecuteScriptAsync(script2, args.Node.Tag);

                // Display the code in the TextBox control
                string[] codes = result.Split('\n');
                textBox1.Text = "HTML code:\n" + codes[0] + "\n\n";
                textBox1.Text += "CSS styles:\n" + codes[1] + "\n\n";
                textBox1.Text += "Attributes:\n" + codes[2] + "\n";
            };
        }

        private void PopulateTreeView(Html
