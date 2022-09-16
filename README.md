# Chartkick groupdate

To quickly create graphs in Rails I would suggest to use gem
https://github.com/ankane/chartkick It is a wrapper for chart.js, highcarts and
goole charts. There is also [js version](https://github.com/ankane/chartkick.js)
It works nice with <https://github.com/ankane/groupdate> for grouping

Let's use form object:
TODO: move to form

```
# app/forms/
class DemographicForm
  include ActiveModel::Model

  GRAPHS = %w[
    MemberProfile.country
    Registration.landing_path
    Registration.source
    Registration.referrer
    Registration.completed_step
  ].freeze

  FIELDS = %i[start_end_date target_graph].freeze
  attr_accessor(:start_date, :end_date, *FIELDS)

  validates(*FIELDS, presence: true)

  def initialize(attributes)
    super(attributes)
    return if start_end_date.blank?

    start_date_string, end_date_string = start_end_date.split(Const.date_separator)
    self.start_date = Date.parse(start_date_string) unless start_date.is_a?(Date)
    self.end_date = Date.parse(end_date_string) unless end_date.is_a?(Date)
  rescue Date::Error # rubocop:todo Lint/SuppressedException
  end

  # https://github.com/ankane/chartkick
  def data
    klass, method = target_graph.split(".")
    klass
      .constantize
      .where(created_at: start_date..end_date)
      .group(method)
      .count
      .sort_by { |_k, v| v }
      .reverse
      .each_with_object({}) do |(value, count), obj|
        limited_label = value.to_s.first(100)
        obj["#{limited_label} (#{count})"] = count
      end
  end
end
```


TODO: options from https://www.sitepoint.com/graphs-on-rails-chartkick-in-practice/

Problem is I do not know how to
* show sum instead of count https://github.com/ankane/groupdate/issues/75
* show only if nested association exists, `joins` will multiply with the
  number of associated rows.

  for error like `lib/patches/db/pg.rb:51:in prepare': PG::UndefinedFunction: ERROR:  could not identify an equality operator for type json (ActiveRecord::StatementInvalid)`
you should change `json` to `jsonb` column since distinct does not know how to
compare json.
